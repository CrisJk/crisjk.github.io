---
layout: post
title: Hadoop实现LeftJoin
categories: Hadoop
tags: 
author: Kuang
---


我在上一篇博客[Hadoop实现LeftJoin操作][1]上已经分享过一种实现LeftJoin操作的方法。这次分享一种自定义数据类型来实现LeftJoin，该方法相对与之前的方法要更高效。简单来说，之前分享的方法是把两张表先按照同一种格式去map，也就是说无论是员工表还是公司表都是一样处理，只不过在Reducer时进行判断，将原本为null的内容替换，所以会有不少的浪费。而本次介绍的方法，自定义一种数据类型，对同一个公司，把公司表放在员工表前面，这样每个Key的第一个一定是公司，不需要做判断，也不需要重复存储一个公司。





说了这么多废话，接下来正式介绍本方法。首先将测试数据放在这方便理解：
employee
jd,david
jd,mike
tb,mike
tb,lucifer
elong,xiaoming
elong,ali
tengxun,xiaoming
tengxun,lilei
xxx,aaa

salary
jd,1600
tb,1800
elong,2000
tengxun,2200

首先定义一个自定义的键值，该键值作为Map过程的输出键值，包含一个Cid，表示公司编号(在这里就是公司名称，后面不再赘述），一个isEmployee,代表是否为employee。hadoop规定，若是自定义键值类型，必须:
1、实现Writable接口
实现readFields,将输入流字节反序列化

```java
public void readFields(DataInput in) throws IOException {
		this.cid = in.readUTF() ;
		this.isEmployee = in.readBoolean();
		
	}
```
实现write,把每个对象序列化到输出

```java
public void write(DataOutput out) throws IOException {
		out.writeUTF(cid);
		out.writeBoolean(isEmployee);
		
	}
```

注意readFields和write里面的顺序必须相同 

２、重写compareTo函数
重写compareTo函数，自定义比较函数


```java
public int compareTo(EmployeeKey o) {
		if(this.cid .equals(o.cid)){
			if(this.getIs()==o.getIs()){
				return 0 ;
			}
			else if(this.getIs()==true){
				return 1 ;
			}
			else return -1 ;
		}
		else {
			return (this.getCid().compareTo(o.getCid())>0)?1:-1 ;
		}
	}
```
3、重写group
如果不重写group，由于公司表和职员表里面的isEmployee不同，同一个公司的公司表和职员表将不在一个List中，这当然不是我们想要的结果。

因此要实现一个继承WritableComparator的类，并在主类中设置setGroupingComparatorClass


```java
class GroupComParator extends WritableComparator{
	public GroupComParator(){
		super(EmployeeKey.class,true) ;
	}
	
	@SuppressWarnings("rawtypes")
	@Override
	public int compare(WritableComparable a , WritableComparable b) {
		EmployeeKey aKey = (EmployeeKey) a ;
		EmployeeKey bKey = (EmployeeKey) b ;
		if(aKey.getCid().equalsIgnoreCase(bKey.getCid())){
			return 0 ;
		}
		else{
			return ( aKey.getCid().compareTo(bKey.getCid()) == -1)?1:-1 ;
		}
	}	
} 
```
自定义好mapper的输出键类型后，就执行mapper操作


```java
class MyMapper extends Mapper<Object,Text,EmployeeKey,Employee> {
	protected void map(Object key,Text value ,Context context)throws IOException,InterruptedException{
		String filePath = ( (FileSplit)context.getInputSplit()).getPath().toString()  ;
		String val = value.toString() ;
		if(val == null || val == "") return ;
		EmployeeKey outKey = new EmployeeKey() ;
		if(filePath.indexOf("employee")!=-1){
			String[] line = val.split(",") ;
			if(line.length<2)return ;
			Employee em = new Employee() ;
			outKey.setCid(line[0]);
			outKey.setIs(true);
			em.setCid(line[0]);
			em.setEid(line[1]);
			em.setSalary("-1");
			context.write(outKey,em);
		}
		else if(filePath.indexOf("salary")!=-1){
			String[] line = val.split(",") ;
			if(line.length<2) return ;
			Employee em = new Employee() ;
			outKey.setCid(line[0]); 
			outKey.setIs(false);
			em.setCid(line[0]) ;
			em.setEid("-1");
			em.setSalary(line[1]) ;
			context.write(outKey,em);
		}
	}
}
```

以上过程会将同一个cid的放在一个list里面，接下来只要在reducer过程中将第一个元素的salary填入其他的元素中即可合并两张表。

```java
package join;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.Writable;
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;


class EmployeeKey implements WritableComparable<EmployeeKey>{

	private String cid ;
	private boolean isEmployee ;
	
	public EmployeeKey()
	{
		
	}
	public EmployeeKey(String cid ,boolean isEmployee){
		this.cid = cid ;
		this.isEmployee = isEmployee;
	}
	
	public void readFields(DataInput in) throws IOException {
		this.cid = in.readUTF() ;
		this.isEmployee = in.readBoolean();
		
	}


	public void write(DataOutput out) throws IOException {
		out.writeUTF(cid);
		out.writeBoolean(isEmployee);
		
	}

	
	public int compareTo(EmployeeKey o) {
		if(this.cid .equals(o.cid)){
			if(this.getIs()==o.getIs()){
				return 0 ;
			}
			else if(this.getIs()==true){
				return 1 ;
			}
			else return -1 ;
		}
		else {
			return (this.getCid().compareTo(o.getCid())>0)?1:-1 ;
		}
	}
	
	public String getCid(){
		return this.cid ;
	}
	public boolean getIs(){
		return this.isEmployee ;
	}
	public void setCid(String cid){
		this.cid = cid;
	}
	public void setIs(boolean isEmployee){
		this.isEmployee  = isEmployee;
	}
	
}

class Employee implements Writable{
	
	private String cid ;
	private String eid ;
	private String salary ;

	public void readFields(DataInput in) throws IOException {
		this.cid = in.readUTF() ;
		this.eid = in.readUTF() ;
		this.salary = in.readUTF() ;
	}

	public void write(DataOutput out) throws IOException {
		out.writeUTF(cid) ;
		out.writeUTF(eid) ;
		out.writeUTF(salary);
		
	}

	public String getCid() {
		return cid;
	}

	public void setCid(String cid) {
		this.cid = cid;
	}

	public String getEid() {
		return eid;
	}

	public void setEid(String eid) {
		this.eid = eid;
	}

	public String getSalary() {
		return salary;
	}

	public void setSalary(String salary) {
		this.salary = salary;
	}
		
	
}


class GroupComParator extends WritableComparator{
	public GroupComParator(){
		super(EmployeeKey.class,true) ;
	}
	
	@SuppressWarnings("rawtypes")
	@Override
	public int compare(WritableComparable a , WritableComparable b) {
		EmployeeKey aKey = (EmployeeKey) a ;
		EmployeeKey bKey = (EmployeeKey) b ;
		if(aKey.getCid().equalsIgnoreCase(bKey.getCid())){
			return 0 ;
		}
		else{
			return ( aKey.getCid().compareTo(bKey.getCid()) == -1)?1:-1 ;
		}
	}	
} 

class MyMapper extends Mapper<Object,Text,EmployeeKey,Employee> {
	protected void map(Object key,Text value ,Context context)throws IOException,InterruptedException{
		String filePath = ( (FileSplit)context.getInputSplit()).getPath().toString()  ;
		String val = value.toString() ;
		if(val == null || val == "") return ;
		EmployeeKey outKey = new EmployeeKey() ;
		if(filePath.indexOf("employee")!=-1){
			String[] line = val.split(",") ;
			if(line.length<2)return ;
			Employee em = new Employee() ;
			outKey.setCid(line[0]);
			outKey.setIs(true);
			//System.out.println(line[1]+" outkey");
			em.setCid(line[0]);
			em.setEid(line[1]);
			em.setSalary("-1");
			context.write(outKey,em);
		}
		else if(filePath.indexOf("salary")!=-1){
			String[] line = val.split(",") ;
			if(line.length<2) return ;
			Employee em = new Employee() ;
			outKey.setCid(line[0]); 
			outKey.setIs(false);
			em.setCid(line[0]) ;
			em.setEid("-1");
			em.setSalary(line[1]) ;
			context.write(outKey,em);
		}
		
		
	}
}

class MyReducer extends Reducer<EmployeeKey,Employee,Object,Text> {
	protected void reduce(EmployeeKey key,Iterable<Employee> vals ,Context context)throws IOException,InterruptedException{
		EmployeeKey eKey = key ;
		String salary="" ;
		for(Employee val : vals){			
			if(eKey.getIs() == true){
				val.setSalary(salary);
				if(salary!="")
				context.write( null,new Text(val.getCid()+" "+val.getEid()+" "+val.getSalary()));
				else
				context.write( null,new Text(val.getCid()+" "+val.getEid()+" "+"NULL"));
			}
			else{
				salary =  new String(val.getSalary());
			}
		}
	}
}
public class LeftJoin {
	
	public static void main(String[] args) throws Exception{
		Configuration config = new Configuration();
		Job job = new Job(config,"Left Join") ;
		FileInputFormat.setInputPaths(job, new Path(args[0])) ;
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		job.setGroupingComparatorClass(GroupComParator.class);
		job.setJarByClass(LeftJoin.class);
		job.setMapperClass(MyMapper.class);
		job.setMapOutputKeyClass(EmployeeKey.class);
		job.setMapOutputValueClass(Employee.class);
	
		job.setReducerClass(MyReducer.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);

		System.exit(job.waitForCompletion(true)?0:1);
		
		
	}
}

```

  [1]: https://consege.github.io/KuangJuncs/blog/Hadoop-LeftJoin.html
