---
layout: post
title: Hadoop Invert Index
categories: Hadoop
tags: 
author: Kuang
description: 
---


倒排索引是文档检索系统中最常见的数据结构，被广泛用于全文索引引擎。它主要是用来存储某个单词(或词组）在一个文档或一组文档那该的存储位置的映射，即提供了一种根据内容来查找文档的方式。由于不是根据文档来确定文档所包含的内容，而是进行了相反的操作（即根据关键字来查找文档），故称为倒排索引。








## 源码


```java
import java.io.IOException;  
import java.util.StringTokenizer;  
  
import org.apache.hadoop.conf.Configuration;  
import org.apache.hadoop.fs.Path;  
import org.apache.hadoop.io.Text;  
import org.apache.hadoop.mapreduce.Job;  
import org.apache.hadoop.mapreduce.Mapper;  
import org.apache.hadoop.mapreduce.Reducer;  
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;  
import org.apache.hadoop.mapreduce.lib.input.FileSplit;  
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;  
  
public class InversedIndex {  
      
    /** 
     * 将输入文件拆分， 
     * 将关键字和关键字所在的文件名作为map的key输出， 
     * 该组合的频率作为value输出 
     * */  
      
    public static class InversedIndexMapper extends Mapper<Object, Text, Text, Text> {  
          
        private Text outKey = new Text();  
        private Text outVal = new Text();  
          
        @Override  
        public void map (Object key,Text value,Context context) {  
            StringTokenizer tokens = new StringTokenizer(value.toString());  
            FileSplit split = (FileSplit) context.getInputSplit();  
            while(tokens.hasMoreTokens()) {  
                String token = tokens.nextToken();  
                try {  
                    outKey.set(token + ":" + split.getPath());  
                    outVal.set("1");  
                    context.write(outKey, outVal);  
                } catch (IOException e) {  
                    e.printStackTrace();  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
    }  
      
    /** 
     * map的输出进入到combiner阶段，此时来自同一个文件的相同关键字进行一次reduce处理， 
     * 将输入的key拆分成关键字和文件名，然后关键字作为输出key， 
     * 将文件名与词频拼接，作为输出value， 
     * 这样就形成了一个关键字，在某一文件中出现的频率的 key--value 对 
     * */  
    public static class InversedIndexCombiner extends Reducer<Text, Text, Text, Text> {  
          
        private Text outKey = new Text();  
        private Text outVal = new Text();  
          
        @Override  
        public void reduce(Text key,Iterable<Text> values,Context context) {  
            String[] keys = key.toString().split(":");  
            int sum = 0;  
            for(Text val : values) {  
                sum += Integer.parseInt(val.toString());  
            }  
            try {  
                outKey.set(keys[0]);  
                int index = keys[keys.length-1].lastIndexOf('/');  
                outVal.set(keys[keys.length-1].substring(index+1) + ":" + sum);  
                context.write(outKey, outVal);  
            } catch (IOException e) {  
                e.printStackTrace();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
          
    }  
      
    /** 
     * 将combiner后的key value对进行reduce， 
     * 由于combiner之后，一个关键字可能对应了多个value，故需要将这些value进行合并输出 
     * */  
      
    public static class InversedIndexReducer extends Reducer<Text, Text, Text, Text> {  
          
        @Override  
        public void reduce (Text key,Iterable<Text> values,Context context) {  
            StringBuffer sb = new StringBuffer();  
            for(Text text : values) {  
                sb.append(text.toString() + " ,");  
            }  
            try {  
                context.write(key, new Text(sb.toString()));  
            } catch (IOException e) {  
                e.printStackTrace();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
      
    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {  
        Configuration conf = new Configuration();  
        Job job = new Job(conf,"index inversed");  
          
        job.setJarByClass(InversedIndex.class);  
        job.setMapperClass(InversedIndexMapper.class);  
        job.setCombinerClass(InversedIndexCombiner.class);  
        job.setReducerClass(InversedIndexReducer.class);  
        job.setMapOutputKeyClass(Text.class);  
        job.setMapOutputValueClass(Text.class);  
        job.setOutputKeyClass(Text.class);  
        job.setOutputValueClass(Text.class);  
  
        job.setNumReduceTasks(3);  
          
        FileInputFormat.addInputPath(job, new Path("input"));  
        FileOutputFormat.setOutputPath(job, new Path("output"));  
          
        System.exit(job.waitForCompletion(true)?0:1);  
          
    }  
  
} 
```

## Example

原文本文件

text1.txt
MapReduce is sample

text2.txt
MapReduce is powerful is sample

text3.txt
Hello MapReduce hello world

<span id = "anchor1" >
运行结果文件

Hello text3.txt:1
MapReduce text3.txt:1 text1.txt:1 , text2.txt:1
hello text2.txt:2 , text1.txt:1
powerful text2.txt:1 ,text1.txt:1
world text3.txt:1

</span>
##过程分析
Map过程：
将输入文件关键字和关键字所在的文件名组合作为map的key输出，map的value为１
如 MapReduce这个关键字map后结果为：
< MapReduce:text1.tx1 ,1>
< MapReduce:text2.txt ,1>
< MapReduce:text3.txt ,1>

Combiner过程:
对Map过程中产生的key相同的做Reduce处理
如< hello:text3.txt , 1> < hello:text3.txt ,1 >合并成< hello　，text3.txt：2>

Reduce过程：

对于combiner过程中产生的具有相同的key的value进行合并输出，最终结果为[运行结果文件](#anchor1)

[摘自Hadoop之道--MapReduce简单应用倒排索引(InvertedIndex) ][1]


  [1]: http://blog.csdn.net/geekcome/article/details/9024419
