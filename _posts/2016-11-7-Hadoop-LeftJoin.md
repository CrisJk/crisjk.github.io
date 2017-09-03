---
layout: post
title: Hadoop MapReduce实现Left Join操作
categories: Hadoop
tags: 
author: Kuang
---
﻿

数据库的Left Join操作就不解释了，参考[图解SQL的各种连接操作][1]
下面来说说如何用hadoop的MapReducer实现数据库的LeftJoin。其实这是个非常简单的过程，举例说明：






假设有两个表employee和salary
Employee
companyId  Employee
jd,         david
jd,         mike
tb,         mike
tb,         lucifer
elong,      xiaoming
elong,      ali
tengxun,    xiaoming
tengxun,    lilei
xxx,        aaa

Salary
companyId  salary
jd,         1600
tb,         1800
elong,      2000
tengxun,    2200

求 leftjoin Employee.companyId == Salary.companyId

假设这两个表以txt存储，分别为employee.txt, salary.txt。
mapper阶段要做的事情很简单，只要把companyId作为map的输出的key,判断当前是哪个文件，value为"a"+employee或"b"+salary。
reducer阶段只需要对同一个key中的value分类，将"a"开头的value放在vecA里面，"b"开头的value放在vecB里面，然后vecA和vecB中就是同一个公司的员工和薪水，注意处理每个不同的key时要把vecA和vecB清空。

至此LeftJoin就做完了。

运行结果：
elong	ali,2000
elong	xiaoming,2000
jd	mike,1600
jd	david,1600
tb	lucifer,1800
tb	mike,1800
tengxun	lilei,2200
tengxun	xiaoming,2200
xxx	aaa,null

可能描述起来有些不清，直接看代码便一目了然：

```java
package org.hadoop.leftJoin;

import java.io.IOException;
import java.util.Vector;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class LeftJoin{

	public static class LeftJoinMapper extends Mapper<Object,Text,Text,Text>{
		protected void map (Object key , Text value , Context context)throws IOException ,InterruptedException{
			String filepath = ((FileSplit)context.getInputSplit()).getPath().toString();
			String val =  value.toString() ;
			if(val == null || val.equals("")) return ;
			if(filepath.indexOf("employee")!=-1){
				String[] line = val.split(",") ;
				if(line.length<2) return ;
				String company_id = line[0];
				String employee = line[1] ;
				context.write(new Text(company_id) ,new Text("a:"+employee) );
			}
			else if (filepath.indexOf("salary")!=-1){
				String[] line = val.split(",") ;
				if(line.length<2)return ;
				String company_id = line[0] ;
				String salary = line[1] ;
				context.write(new Text(company_id) , new Text("b:"+salary)) ;
			}
		}
		
	}
	
	public static class LeftJoinReducer extends Reducer<Text,Text,Text,Text>{
		protected void reduce(Text key ,Iterable<Text> values , Context context) throws IOException , InterruptedException{
			Vector<String> vecA = new Vector<String>() ;
			Vector<String> vecB = new Vector<String>() ;
			for(Text vals:values){
				String val = vals.toString() ;
				if(val.startsWith("a:")){
					vecA.add(val.substring(2)) ;
				}
				else if(val.startsWith("b:")){
					vecB.add(val.substring(2)) ;
				}
			}
			
			for (int i = 0; i < vecA.size(); i++) {
                if (vecB.size() == 0) {
                    context.write(key, new Text(vecA.get(i) + "," + "null"));
                } else {
                    for (int j = 0; j < vecB.size(); j++) {
                        context.write(key, new Text(vecA.get(i) + "," + vecB.get(j)));
                    }
                }
            }
		}
	}
	
	public static void main(String[] args) throws Exception{
		Configuration config = new Configuration() ;
		Job job = Job.getInstance(config, "left join");
		job.setJarByClass(LeftJoin.class);
		job.setMapperClass(LeftJoinMapper.class);
		job.setReducerClass(LeftJoinReducer.class) ;
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class) ;
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);
		
		//job.setNumReduceTasks(3);
		config.set("mapred.textoutputformat.separator", ",");
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
          
        System.exit(job.waitForCompletion(true)?0:1);  
  
	}
}

```
代码和例子来自[hadoop 用MR实现join操作 ][2]


  [1]: http://www.cnblogs.com/sunjie9606/p/4167190.html
  [2]: http://blog.csdn.net/bitcarmanlee/article/details/51863358
