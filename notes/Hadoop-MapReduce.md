# MapReduce--分布式计算框架

<nav>
<a href="#一mapreduce概述">一、MapReduce概述</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#11-mapreduce-简介">1.1 MapReduce 简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#12-mapreduce-核心编程思想">1.2 MapReduce 核心编程思想</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#13-常用数据序列化类型">1.3 常用数据序列化类型</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#14-mapreduce编程规范">1.4 MapReduce编程规范</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#15-wordcount案例实操">1.5 WordCount案例实操</a><br/>
<a href="#二-hadoop序列化">二 、Hadoop序列化</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#21-序列化概述">2.1 序列化概述</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#22-自定义bean对象实现序列化接口writable">2.2 自定义bean对象实现序列化接口（Writable）</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#23-序列化案例-手机流量统计">2.3 序列化案例-手机流量统计</a><br/>
<a href="#三mapreduce框架原理">三、MapReduce框架原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#31--inputformat数据输入">3.1  InputFormat数据输入</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#32-mapreduce工作流程">3.2 MapReduce工作流程</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#33-shuffle机制">3.3 Shuffle机制</a><br/>
    &nbsp;&nbsp;&nbsp;&nbsp;<a href="#34-maptask工作机制">3.4 MapTask工作机制</a><br/>
    &nbsp;&nbsp;&nbsp;&nbsp;<a href="#35-reducetask工作机制">3.5 ReduceTask工作机制</a><br/>
    &nbsp;&nbsp;&nbsp;&nbsp;<a href="#36-outputformat数据输出">3.6 OutputFormat数据输出</a><br/>
    &nbsp;&nbsp;&nbsp;&nbsp;<a href="#37-join多种应用">3.7 Join多种应用</a><br/>
    &nbsp;&nbsp;&nbsp;&nbsp;<a href="#38-计数器应用">3.8 计数器应用</a><br/>
    &nbsp;&nbsp;&nbsp;&nbsp;<a href="#39-数据清洗etl">3.9 数据清洗（ETL）</a><br/>
    &nbsp;&nbsp;&nbsp;&nbsp;<a href="#310-mapreduce开发总结">3.10 MapReduce开发总结</a><br/>
</nav>




## 一、MapReduce概述

### 1.1 MapReduce 简介

1.MapReduce是一个分布式运算程序的编程框架，是用户开发“基于Hadoop的数据分析应用”的核心框架

2.MapReduce的核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个Hadoop集群上。

3.MapReduce优点

（1）MapReduce易于编程：它简单的实现一些接口，就可以完成一个分布式程序，这个分布式程序可以分布到大量廉价的PC机器上运行。也就是说你写一个分布式程序，就跟写一个简单的串行程序是一模一样的。

（2）它还具有良好的扩展：当你计算资源不能得到满足的时候，你可以通过简单的增加机器来扩展它的计算能力

（3）高容错性：一台机器挂了，它可以把上面的计算任务转移到另一个节点上运行，不至于晕个任务失败，此过程不需要人工参与，完全是由Hadoop内部完成的。

（4）PB级以上海量数据的离线处理：可以实现上千台服务器集群并发工作，提供数据处理能力。

4.HDFS不足

（1）不擅长实时计算

（2）不擅长流式计算

（3）不擅长DAG（有向无环图）计算



### 1.2 MapReduce 核心编程思想

![MapReduce1-5](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce1-5.png)

（1）分布式的运算程序往往需要分成至少2个阶段。

（2）第一个阶段的MapTask并发实例，完全并行运行，互不相干。

（3）第二个阶段的ReduceTask并发实例互不相干，但是他们的数据依赖于上一个阶段的所有MapTask并发实例的输出。

（4）MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序，串行运行。

总结：分析WordCount数据流走向深入理解MapReduce核心思想。



### 1.3 常用数据序列化类型

| **Java**类型 | **Hadoop Writable类型** |
| ------------ | ----------------------- |
| Boolean      | BooleanWritable         |
| Byte         | ByteWritable            |
| Int          | IntWritable             |
| Float        | FloatWritable           |
| Long         | LongWritable            |
| Double       | DoubleWritable          |
| **String**   | **Text**                |
| Map          | MapWritable             |
| Array        | ArrayWritable           |



### 1.4 MapReduce编程规范

用户编写的程序分成三个部分：Mapper、Reducer和Driver

**1.Mapper阶段**

（1）用户自定义的Mapper要继承自己的父类

（2）Mapper的输入数据是KV对的形式（KV的类型可自定义）

（3）Mapper中的业务逻辑写在map（）方法中

（4）Mapper的输出数据是KV对的形式（KV的类型可自定义）

（5）map()方法（MapTask进程）对每一个<K，V>调用一次

**2.Reducer阶段**

（1）用户自定义的Reducer要继承自己的父类

（2）Reducer的输入数据类型对应Mapper的输出数据类型，也是KV

（3）Reducer的业务逻辑写在rebuce()方法中

（4）ReduceTask进程对每一组相同k的<k,v>组调用一次reduce()方法

**3.Driver阶段**

相录于YARN集群的客户端，用于提交我们整个程序到YARN集群，提交的是封装了MapReduce程序相关运行参数的job对象

### 1.5 WordCount案例实操

**1.需求**

在给定的文本文件中统计输出每一个单词出现的总次数

(1) 自己创建一个hello.txt,添加如下数据

~~~shell
aa aa
ss ss
cls cls
jiao
banzhang
xue
hadoop
~~~

(2) 期望输出数据

~~~shell
aa	2
banzhang	1
cls	2
hadoop	1
jiao	1
ss	2
xue	1
~~~



**2.环境准备**

(1) 创建maven工程

(2) 在pom.xml文件中添加如下依赖

~~~java
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>2.12.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>3.1.3</version>
    </dependency>
</dependencies>
~~~

(3) 在项目的src/main/resources目录下，新建一个文件，命名为“log4j2.xml”，在文件中填入

~~~java
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="error" strict="true" name="XMLConfig">
    <Appenders>
        <!-- 类型名为Console，名称为必须属性 -->
        <Appender type="Console" name="STDOUT">
            <!-- 布局为PatternLayout的方式，
            输出样式为[INFO] [2018-01-22 17:34:01][org.test.Console]I'm here -->
            <Layout type="PatternLayout"
                    pattern="[%p] [%d{yyyy-MM-dd HH:mm:ss}][%c{10}]%m%n" />
        </Appender>

    </Appenders>

    <Loggers>
        <!-- 可加性为false -->
        <Logger name="test" level="info" additivity="false">
            <AppenderRef ref="STDOUT" />
        </Logger>

        <!-- root loggerConfig设置 -->
        <Root level="info">
            <AppenderRef ref="STDOUT" />
        </Root>
    </Loggers>

</Configuration>
~~~



**3.编写程序**

(1) 编写Mapper类

~~~java
package com.cto15.mapreduce;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class WordcountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	
	Text k = new Text();
	IntWritable v = new IntWritable(1);
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
		
		// 1 获取一行
		String line = value.toString();
		
		// 2 切割
		String[] words = line.split(" ");
		
		// 3 输出
		for (String word : words) {
			
			k.set(word);
			context.write(k, v);
		}
	}
}
~~~

(2) 编写Reducer类

~~~java
package com.cto15.mapreduce.wordcount;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordcountReducer extends Reducer<Text, IntWritable, Text, IntWritable>{

int sum;
IntWritable v = new IntWritable();

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {
		
		// 1 累加求和
		sum = 0;
		for (IntWritable count : values) {
			sum += count.get();
		}
		
		// 2 输出
       v.set(sum);
		context.write(key,v);
	}
}
~~~

(3) 编写Driver驱动类

~~~java
package com.cto15.mapreduce.wordcount;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordcountDriver {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

		// 1 获取配置信息以及封装任务
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);

		// 2 设置jar加载路径
		job.setJarByClass(WordcountDriver.class);

		// 3 设置map和reduce类
		job.setMapperClass(WordcountMapper.class);
		job.setReducerClass(WordcountReducer.class);

		// 4 设置map输出
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);

		// 5 设置最终输出kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		// 6 设置输入和输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 7 提交
		boolean result = job.waitForCompletion(true);

		System.exit(result ? 0 : 1);
	}
}
~~~



**4.本地测试**

(1) 需要首先配置好HadoopHome变量以及Windows运行依赖

(2) 在Eclipse/Idea上运行程序



**5.集群上测试**

(1) 用maven打jar包，如果有需要一并打进去的依赖，需要添加打包插件 

注意：<mainClass>com.cto15.mr.WordcountDriver</mainClass>部分需要替换为自己工程主类

~~~java
<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.3.2</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-assembly-plugin </artifactId>
				<configuration>
					<descriptorRefs>
						<descriptorRef>jar-with-dependencies</descriptorRef>
					</descriptorRefs>
					<archive>
						<manifest>
							<mainClass>com.cto15.mr.WordcountDriver</mainClass>
						</manifest>
					</archive>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
~~~

注意：如果工程上显示红叉。在项目上右键->maven->update project即可。

(2) 将程序打成jar包，然后拷贝到Hadoop集群中

步骤详情：Maven ->lifecycle-> install。等待编译完成就会在项目的target文件夹中生成jar包。如果看不到。在项目上右键 -> Refresh，即可看到。修改不带依赖的jar包名称为wc.jar，并拷贝该jar包到Hadoop集群。

(3) 启动Hadoop集群

(4) 执行WordCount程序

~~~shell
[cto15@hadoop102 software]$ hadoop jar  wc.jar
 com.cto15.wordcount.WordcountDriver /user/cto15/input /user/cto15/output
~~~



**6.在Windows上向集群提交任务**

(1) 添加必要配置信息

~~~java
public class WordcountDriver {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

		// 1 获取配置信息以及封装任务
		Configuration configuration = new Configuration();
	    //设置HDFS NameNode的地址
       configuration.set("fs.defaultFS", "hdfs://hadoop102:8020");
       // 指定MapReduce运行在Yarn上
       configuration.set("mapreduce.framework.name","yarn");
       // 指定mapreduce可以在远程集群运行
		configuration.set("mapreduce.app-submission.cross-platform","true");
    
	//指定Yarn resourcemanager的位置
	configuration.set("yarn.resourcemanager.hostname","hadoop103");

		Job job = Job.getInstance(configuration);

		// 2 设置jar加载路径
		job.setJarByClass(WordcountDriver.class);

		// 3 设置map和reduce类
		job.setMapperClass(WordcountMapper.class);
		job.setReducerClass(WordcountReducer.class);

		// 4 设置map输出
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);

		// 5 设置最终输出kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		// 6 设置输入和输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 7 提交
		boolean result = job.waitForCompletion(true);

		System.exit(result ? 0 : 1);
	}
}
~~~

(2) 先进行打包，并将Jar包设置到Driver中，集群中运行需要指定jar包

~~~java
public class WordcountDriver {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

		// 1 获取配置信息以及封装任务
		Configuration configuration = new Configuration();

       configuration.set("fs.defaultFS", "hdfs://hadoop102:8020");
       configuration.set("mapreduce.framework.name","yarn");
       configuration.set("mapreduce.app-submission.cross-platform","true");
    configuration.set("yarn.resourcemanager.hostname","hadoop103");

		Job job = Job.getInstance(configuration);

		// 2 设置jar加载路径
		job.setJar("D:\\input\\MapReduce-1.0-SNAPSHOT.jar");

		// 3 设置map和reduce类
		job.setMapperClass(WordcountMapper.class);
		job.setReducerClass(WordcountReducer.class);

		// 4 设置map输出
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);

		// 5 设置最终输出kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		// 6 设置输入和输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 7 提交
		boolean result = job.waitForCompletion(true);

		System.exit(result ? 0 : 1);
	}
}
~~~

(3) 编辑任务配置

![MapReduce1-10](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce1-10.png)

(4) 提交并在集群查看结果



## 二 、Hadoop序列化

### 2.1 序列化概述

![MapReduce2-1](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce2-1.png)

![MapReduce2-2](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce2-2.png)



### 2.2 自定义bean对象实现序列化接口（Writable）

在企业开发中往往常用的基本序列化类型不能满足所有需求，比如在Hadoop框架内部传递一个bean对象，那么该对象就需要实现序列化接口。

具体实现bean对象序列化步骤如下7步：

（1）必须实现Writable接口

（2）反序列化时，需要反射调用空参构造函数，所以必须有空参构造

~~~java
public FlowBean() {
	super();
}
~~~

（3）重写序列化方法

~~~java
@Override
public void write(DataOutput out) throws IOException {
	out.writeLong(upFlow);
	out.writeLong(downFlow);
	out.writeLong(sumFlow);
}
~~~

（4）重写反序列化方法

```java
@Override
public void readFields(DataInput in) throws IOException {
	upFlow = in.readLong();
	downFlow = in.readLong();
	sumFlow = in.readLong();
}
```

（5）注意反序列化的顺序和序列化的顺序完全一致

（6）要想把结果显示在文件中，需要重写toString()，可用”\t”分开，方便后续用。

（7）如果需要将自定义的bean放在key中传输，则还需要实现Comparable接口，因为MapReduce框中的Shuffle过程要求对key必须能排序。详见后面排序案例。

```java
@Override
public int compareTo(FlowBean o) {
	// 倒序排列，从大到小
	return this.sumFlow > o.getSumFlow() ? -1 : 1;
}
```



### 2.3 序列化案例-手机流量统计

**1.需求**

统计每一个手机号耗费的总上行流量、下行流量、总流量

（1）输入数据 phone_data.txt

~~~
1	13736230513	192.196.100.1	www.cto15.com	2481	24681	200
2	13846544121	192.196.100.2			264	0	200
3 	13956435636	192.196.100.3			132	1512	200
4 	13966251146	192.168.100.1			240	0	404
5 	18271575951	192.168.100.2	www.cto15.com	1527	2106	200
6 	84188413	192.168.100.3	www.cto15.com	4116	1432	200
7 	13590439668	192.168.100.4			1116	954	200
8 	15910133277	192.168.100.5	www.hao123.com	3156	2936	200
9 	13729199489	192.168.100.6			240	0	200
10 	13630577991	192.168.100.7	www.shouhu.com	6960	690	200
11 	15043685818	192.168.100.8	www.baidu.com	3659	3538	200
12 	15959002129	192.168.100.9	www.cto15.com	1938	180	500
13 	13560439638	192.168.100.10			918	4938	200
14 	13470253144	192.168.100.11			180	180	200
15 	13682846555	192.168.100.12	www.qq.com	1938	2910	200
16 	13992314666	192.168.100.13	www.gaga.com	3008	3720	200
17 	13509468723	192.168.100.14	www.qinghua.com	7335	110349	404
18 	18390173782	192.168.100.15	www.sogou.com	9531	2412	200
19 	13975057813	192.168.100.16	www.baidu.com	11058	48243	200
20 	13768778790	192.168.100.17			120	120	200
21 	13568436656	192.168.100.18	www.alibaba.com	2481	24681	200
22 	13568436656	192.168.100.19			1116	954	200
~~~

（2）输入数据格式：

```
7 	13560436666	120.196.100.99		1116		 954			200
id	手机号码		网络ip			上行流量  下行流量     网络状态码
```

（3）期望输出数据格式

```
13560436666 		1116		      954 			2070
手机号码		    上行流量        下行流量		总流量
```



**2.编写MapReduce程序**

（1）编写流量统计的Bean对象

~~~java
package com.cto15.mapreduce.flowsum;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.Writable;

// 1 实现writable接口
public class FlowBean implements Writable{

	private long upFlow;
	private long downFlow;
	private long sumFlow;
	
	//2  反序列化时，需要反射调用空参构造函数，所以必须有
	public FlowBean() {
		super();
	}

	public FlowBean(long upFlow, long downFlow) {
		super();
		this.upFlow = upFlow;
		this.downFlow = downFlow;
		this.sumFlow = upFlow + downFlow;
	}
	
	//3  写序列化方法
	@Override
	public void write(DataOutput out) throws IOException {
		out.writeLong(upFlow);
		out.writeLong(downFlow);
		out.writeLong(sumFlow);
	}
	
	//4 反序列化方法
	//5 反序列化方法读顺序必须和写序列化方法的写顺序必须一致
	@Override
	public void readFields(DataInput in) throws IOException {
		this.upFlow  = in.readLong();
		this.downFlow = in.readLong();
		this.sumFlow = in.readLong();
	}

	// 6 编写toString方法，方便后续打印到文本
	@Override
	public String toString() {
		return upFlow + "\t" + downFlow + "\t" + sumFlow;
	}

	public long getUpFlow() {
		return upFlow;
	}

	public void setUpFlow(long upFlow) {
		this.upFlow = upFlow;
	}

	public long getDownFlow() {
		return downFlow;
	}

	public void setDownFlow(long downFlow) {
		this.downFlow = downFlow;
	}

	public long getSumFlow() {
		return sumFlow;
	}

	public void setSumFlow(long sumFlow) {
		this.sumFlow = sumFlow;
	}
}
~~~

（2）编写Mapper类

```java
package com.cto15.mapreduce.flowsum;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class FlowCountMapper extends Mapper<LongWritable, Text, Text, FlowBean>{
	
	FlowBean v = new FlowBean();
	Text k = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
		
		// 1 获取一行
		String line = value.toString();
		
		// 2 切割字段
		String[] fields = line.split("\t");
		
		// 3 封装对象
		// 取出手机号码
		String phoneNum = fields[1];

		// 取出上行流量和下行流量
		long upFlow = Long.parseLong(fields[fields.length - 3]);
		long downFlow = Long.parseLong(fields[fields.length - 2]);

		k.set(phoneNum);
		v.set(downFlow, upFlow);
		
		// 4 写出
		context.write(k, v);
	}
}
```

（3）编写Reducer类

```java
package com.cto15.mapreduce.flowsum;
import java.io.IOException;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class FlowCountReducer extends Reducer<Text, FlowBean, Text, FlowBean> {

	@Override
	protected void reduce(Text key, Iterable<FlowBean> values, Context context)throws IOException, InterruptedException {

		long sum_upFlow = 0;
		long sum_downFlow = 0;

		// 1 遍历所用bean，将其中的上行流量，下行流量分别累加
		for (FlowBean flowBean : values) {
			sum_upFlow += flowBean.getUpFlow();
			sum_downFlow += flowBean.getDownFlow();
		}

		// 2 封装对象
		FlowBean resultBean = new FlowBean(sum_upFlow, sum_downFlow);
		
		// 3 写出
		context.write(key, resultBean);
	}
}
```

（4）编写Driver驱动类

```java
package com.cto15.mapreduce.flowsum;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class FlowsumDriver {

	public static void main(String[] args) throws IllegalArgumentException, IOException, ClassNotFoundException, InterruptedException {
		
// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
args = new String[] { "e:/input/inputflow", "e:/output1" };

		// 1 获取配置信息，或者job对象实例
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);

		// 6 指定本程序的jar包所在的本地路径
		job.setJarByClass(FlowsumDriver.class);

		// 2 指定本业务job要使用的mapper/Reducer业务类
		job.setMapperClass(FlowCountMapper.class);
		job.setReducerClass(FlowCountReducer.class);

		// 3 指定mapper输出数据的kv类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(FlowBean.class);

		// 4 指定最终输出的数据的kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(FlowBean.class);
		
		// 5 指定job的输入原始文件所在目录
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```



## 三、MapReduce框架原理

### 3.1  InputFormat数据输入

![MapReduce3-1](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-1.png)



**3.1.1 切片与MapTask并行度决定机制**

1.问题引出

MapTask的并行度决定Map阶段的任务处理并发度，进而影响到整个Job的处理速度。

思考：1G的数据，启动8个MapTask，可以提高集群的并发处理能力。那么1K的数据，也启动8个MapTask，会提高集群性能吗？MapTask并行任务是否越多越好呢？哪些因素影响了MapTask并行度？

2．MapTask并行度决定机制

**数据块：**Block是HDFS物理上把数据分成一块一块。

**数据切片：**数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。

MapTask并行度决定机制图：

![MapReduce3-2](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-2.png)

**3.1.2 Job提交流程源码和切片源码详解**

1.Job提交流程源码详解

~~~java
waitForCompletion()

submit();

// 1建立连接
	connect();	
		// 1）创建提交Job的代理
		new Cluster(getConfiguration());
			// （1）判断是本地yarn还是远程
			initialize(jobTrackAddr, conf); 

// 2 提交job
submitter.submitJobInternal(Job.this, cluster)
	// 1）创建给集群提交数据的Stag路径
	Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);

	// 2）获取jobid ，并创建Job路径
	JobID jobId = submitClient.getNewJobID();

	// 3）拷贝jar包到集群
copyAndConfigureFiles(job, submitJobDir);	
	rUploader.uploadFiles(job, jobSubmitDir);

// 4）计算切片，生成切片规划文件
writeSplits(job, submitJobDir);
		maps = writeNewSplits(job, jobSubmitDir);
		input.getSplits(job);

// 5）向Stag路径写XML配置文件
writeConf(conf, submitJobFile);
	conf.writeXml(out);

// 6）提交Job,返回提交状态
status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());
~~~

Job提交流程源码分析图：

![MapReduce3-3](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-3.png)

2．FileInputFormat切片源码解析(input.getSplits(job))

FileInputFormat切片机制图：

![MapReduce3-4](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-4.png)



**3.1.3 FileInputFormat切片机制**

FileInputFormat切片机制：

![MapReduce3-5](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-5.png)

FileInputFormat切片大小参数配置：

![MapReduce3-6](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-6.png)



**3.1.4 FileInputFormat实现类**

![MapReduce3-7](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-7.png)

![MapReduce3-8](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-8.png)

KeyValueTextInputFormat：

![MapReduce3-9](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-9.png)

NLineInputFormat：

![MapReduce3-10](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-10.png)



**3.1.5 CombineTextInputFormat切片机制**

框架默认的TextInputFormat切片机制是对任务按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个MapTask，这样如果有大量小文件，就会产生大量的MapTask，处理效率极其低下

1、应用场景：

CombineTextInputFormat用于小文件过多的场景，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个MapTask处理。

2、虚拟存储切片最大值设置

CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m

注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

3、切片机制

生成切片过程包括：虚拟存储过程和切片过程二部分。

CombineTextInputFormat切片机制：

![MapReduce3-11](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-11.png)

（1）虚拟存储过程：

将输入目录下所有文件大小，依次和设置的setMaxInputSplitSize值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值2倍，此时将文件均分成2个虚拟存储块（防止出现太小切片）。

例如setMaxInputSplitSize值为4M，输入文件大小为8.02M，则先逻辑上分成一个4M。剩余的大小为4.02M，如果按照4M逻辑划分，就会出现0.02M的小的虚拟存储文件，所以将剩余的4.02M文件切分成（2.01M和2.01M）两个文件。

（2）切片过程：

（a）判断虚拟存储的文件大小是否大于setMaxInputSplitSize值，大于等于则单独形成一个切片。

（b）如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。

（c）测试举例：有4个小文件大小分别为1.7M、5.1M、3.4M以及6.8M这四个小文件，则虚拟存储之后形成6个文件块，大小分别为：

1.7M，（2.55M、2.55M），3.4M以及（3.4M、3.4M）

最终会形成3个切片，大小分别为：

（1.7+2.55）M，（2.55+3.4）M，（3.4+3.4）M



**3.1.6 CombineTextInputFormat案例实操**

**1．需求**

将输入的大量小文件合并成一个切片统一处理。

（1）输入数据

准备4个小文件

（2）期望

期望一个切片处理4个文件

**2．实现过程**

（1）不做任何处理，运行1.6**节的WordCount**案例程序，观察切片个数为4。

~~~
number of splits:4
~~~

（2）在WordcountDriver中增加如下代码，运行程序，并观察运行的切片个数为3。

（a）驱动类中添加代码如下：

```
// 如果不设置InputFormat，它默认用的是TextInputFormat.class
job.setInputFormatClass(CombineTextInputFormat.class);

//虚拟存储切片最大值设置4m
CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);
```

（b）运行如果为3个切片

~~~
number of splits:3
~~~

（3）在WordcountDriver中增加如下代码，运行程序，并观察运行的切片个数为1

（a）驱动中添加代码如下：

```
// 如果不设置InputFormat，它默认用的是TextInputFormat.class
job.setInputFormatClass(CombineTextInputFormat.class);

//虚拟存储切片最大值设置20m
CombineTextInputFormat.setMaxInputSplitSize(job, 20971520);
```

（b）运行如果为1个切片。

```
number of splits:1
```



**3.1.7 KeyValueTextInputFormat使用案例**

**1．需求**

统计输入文件中每一行的第一个单词相同的行数。

（1）输入数据

```
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
```

（2）期望结果数据

```
banzhang	2
xihuan	2
```

**2．需求分析**

![MapReduce3-12](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-12.png)

**3．代码实现**

（1）编写Mapper类

```java
package com.cto15.mapreduce.KeyValueTextInputFormat;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class KVTextMapper extends Mapper<Text, Text, Text, LongWritable>{
	
// 1 设置value
   LongWritable v = new LongWritable(1);  
    
	@Override
	protected void map(Text key, Text value, Context context)
			throws IOException, InterruptedException {

// banzhang ni hao
        
        // 2 写出
        context.write(key, v);  
	}
}
```

（2）编写Reducer类

```java
package com.cto15.mapreduce.KeyValueTextInputFormat;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class KVTextReducer extends Reducer<Text, LongWritable, Text, LongWritable>{
	
    LongWritable v = new LongWritable();  
    
	@Override
	protected void reduce(Text key, Iterable<LongWritable> values,	Context context) throws IOException, InterruptedException {
		
		 long sum = 0L;  

		 // 1 汇总统计
        for (LongWritable value : values) {  
            sum += value.get();  
        }
         
        v.set(sum);  
         
        // 2 输出
        context.write(key, v);  
	}
}
```

（3）编写Driver类

```java
package com.cto15.mapreduce.keyvaleTextInputFormat;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.KeyValueLineRecordReader;
import org.apache.hadoop.mapreduce.lib.input.KeyValueTextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class KVTextDriver {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		
		Configuration conf = new Configuration();
		// 设置切割符
	conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR, " ");
		// 1 获取job对象
		Job job = Job.getInstance(conf);
		
		// 2 设置jar包位置，关联mapper和reducer
		job.setJarByClass(KVTextDriver.class);
		job.setMapperClass(KVTextMapper.class);
job.setReducerClass(KVTextReducer.class);
				
		// 3 设置map输出kv类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(LongWritable.class);

		// 4 设置最终输出kv类型
		job.setOutputKeyClass(Text.class);
job.setOutputValueClass(LongWritable.class);
		
		// 5 设置输入输出数据路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		
		// 设置输入格式
	job.setInputFormatClass(KeyValueTextInputFormat.class);
		
		// 6 设置输出数据路径
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		// 7 提交job
		job.waitForCompletion(true);
	}
}
```



**3.1.8 NLineInputFormat使用案例**

**1．需求**

对每个单词进行个数统计，要求根据每个输入文件的行数来规定输出多少个切片。此案例要求每三行放入一个切片中。

（1）输入数据

```
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang banzhang ni hao
xihuan hadoop banzhang
```

（2）期望输出数据

```
Number of splits:4
```

**2．需求分析**

![MapReduce3-13](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-13.png)

**3．代码实现**

（1）编写Mapper类

```java
package com.cto15.mapreduce.nline;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class NLineMapper extends Mapper<LongWritable, Text, Text, LongWritable>{
	
	private Text k = new Text();
	private LongWritable v = new LongWritable(1);
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
		
		 // 1 获取一行
        String line = value.toString();
        
        // 2 切割
        String[] splited = line.split(" ");
        
        // 3 循环写出
        for (int i = 0; i < splited.length; i++) {
        	
        	k.set(splited[i]);
        	
           context.write(k, v);
        }
	}
}
```

（2）编写Reducer类

```java
package com.cto15.mapreduce.nline;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class NLineReducer extends Reducer<Text, LongWritable, Text, LongWritable>{
	
	LongWritable v = new LongWritable();
	
	@Override
	protected void reduce(Text key, Iterable<LongWritable> values,	Context context) throws IOException, InterruptedException {
		
        long sum = 0l;

        // 1 汇总
        for (LongWritable value : values) {
            sum += value.get();
        }  
        
        v.set(sum);
        
        // 2 输出
        context.write(key, v);
	}
}
```

（3）编写Driver类

```java
package com.cto15.mapreduce.nline;
import java.io.IOException;
import java.net.URISyntaxException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.NLineInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class NLineDriver {
	
	public static void main(String[] args) throws IOException, URISyntaxException, ClassNotFoundException, InterruptedException {
		
// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
args = new String[] { "e:/input/inputword", "e:/output1" };

		 // 1 获取job对象
		 Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration);
        
        // 7设置每个切片InputSplit中划分三条记录
        NLineInputFormat.setNumLinesPerSplit(job, 3);
          
        // 8使用NLineInputFormat处理记录数  
        job.setInputFormatClass(NLineInputFormat.class);  
          
        // 2设置jar包位置，关联mapper和reducer
        job.setJarByClass(NLineDriver.class);  
        job.setMapperClass(NLineMapper.class);  
        job.setReducerClass(NLineReducer.class);  
        
        // 3设置map输出kv类型
        job.setMapOutputKeyClass(Text.class);  
        job.setMapOutputValueClass(LongWritable.class);  
        
        // 4设置最终输出kv类型
        job.setOutputKeyClass(Text.class);  
        job.setOutputValueClass(LongWritable.class);  
          
        // 5设置输入输出数据路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));  
        FileOutputFormat.setOutputPath(job, new Path(args[1]));  
          
        // 6提交job
        job.waitForCompletion(true);  
	}
}
```

**4．测试**

（1）输入数据

```
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang banzhang ni hao
xihuan hadoop banzhang
```

（2）输出结果的切片数，如下图所示：

![MapReduce3-14](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-14.png)



**3.1.9 自定义InputFormat**

![MapReduce3-15](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-15.png)

**3.1.10 自定义InputFormat案例实操**

无论HDFS还是MapReduce，在处理小文件时效率都非常低，但又难免面临处理大量小文件的场景，此时，就需要有相应解决方案。可以自定义InputFormat实现小文件的合并。

**1．需求**

将多个小文件合并成一个SequenceFile文件（SequenceFile文件是Hadoop用来存储二进制形式的key-value对的文件格式），SequenceFile里面存储着多个文件，存储的形式为文件路径+名称为key，文件内容为value。

（1）输入数据

one.txt

```
yongpeng weidong weinan
sanfeng luozong xiaoming
```

two.txt

```
longlong fanfan
mazong kailun yuhang yixin
longlong fanfan
mazong kailun yuhang yixin
```

three.txt

```
shuaige changmo zhenqiang 
dongli lingu xuanxuan
```

（2）期望输出文件格式

part-r-00000



**2．需求分析**

![MapReduce3-16](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-16.png)

**3．程序实现**

（1）自定义InputFromat

```java
package com.cto15.mapreduce.inputformat;
import java.io.IOException;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

// 定义类继承FileInputFormat
public class WholeFileInputformat extends FileInputFormat<Text, BytesWritable>{
	
	@Override
	protected boolean isSplitable(JobContext context, Path filename) {
		return false;
	}

	@Override
	public RecordReader<Text, BytesWritable> createRecordReader(InputSplit split, TaskAttemptContext context)	throws IOException, InterruptedException {
		
		WholeRecordReader recordReader = new WholeRecordReader();
		recordReader.initialize(split, context);
		
		return recordReader;
	}
}
```

（2）自定义RecordReader类

```java
package com.cto15.mapreduce.inputformat;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class WholeRecordReader extends RecordReader<Text, BytesWritable>{

	private Configuration configuration;
	private FileSplit split;
	
	private boolean isProgress= true;
	private BytesWritable value = new BytesWritable();
	private Text k = new Text();

	@Override
	public void initialize(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
		
		this.split = (FileSplit)split;
		configuration = context.getConfiguration();
	}

	@Override
	public boolean nextKeyValue() throws IOException, InterruptedException {
		
		if (isProgress) {

			// 1 定义缓存区
			byte[] contents = new byte[(int)split.getLength()];
			
			FileSystem fs = null;
			FSDataInputStream fis = null;
			
			try {
				// 2 获取文件系统
				Path path = split.getPath();
				fs = path.getFileSystem(configuration);
				
				// 3 读取数据
				fis = fs.open(path);
				
				// 4 读取文件内容
				IOUtils.readFully(fis, contents, 0, contents.length);
				
				// 5 输出文件内容
				value.set(contents, 0, contents.length);

// 6 获取文件路径及名称
String name = split.getPath().toString();

// 7 设置输出的key值
k.set(name);

			} catch (Exception e) {
				
			}finally {
				IOUtils.closeStream(fis);
			}
			
			isProgress = false;
			
			return true;
		}
		
		return false;
	}

	@Override
	public Text getCurrentKey() throws IOException, InterruptedException {
		return k;
	}

	@Override
	public BytesWritable getCurrentValue() throws IOException, InterruptedException {
		return value;
	}

	@Override
	public float getProgress() throws IOException, InterruptedException {
		return 0;
	}

	@Override
	public void close() throws IOException {
	}
}
```

（3）编写SequenceFileMapper类处理流程

```java
package com.cto15.mapreduce.inputformat;
import java.io.IOException;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class SequenceFileMapper extends Mapper<Text, BytesWritable, Text, BytesWritable>{
	
	@Override
	protected void map(Text key, BytesWritable value,			Context context)		throws IOException, InterruptedException {

		context.write(key, value);
	}
}
```

（4）编写SequenceFileReducer类处理流程

```java
package com.cto15.mapreduce.inputformat;
import java.io.IOException;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class SequenceFileReducer extends Reducer<Text, BytesWritable, Text, BytesWritable> {

	@Override
	protected void reduce(Text key, Iterable<BytesWritable> values, Context context)		throws IOException, InterruptedException {

		context.write(key, values.iterator().next());
	}
}
```

（5）编写SequenceFileDriver类处理流程

```java
package com.cto15.mapreduce.inputformat;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.SequenceFileOutputFormat;

public class SequenceFileDriver {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		
       // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
		args = new String[] { "e:/input/inputinputformat", "e:/output1" };

       // 1 获取job对象
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);

       // 2 设置jar包存储位置、关联自定义的mapper和reducer
		job.setJarByClass(SequenceFileDriver.class);
		job.setMapperClass(SequenceFileMapper.class);
		job.setReducerClass(SequenceFileReducer.class);

       // 7设置输入的inputFormat
		job.setInputFormatClass(WholeFileInputformat.class);

       // 8设置输出的outputFormat
	 job.setOutputFormatClass(SequenceFileOutputFormat.class);
       
// 3 设置map输出端的kv类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(BytesWritable.class);
		
       // 4 设置最终输出端的kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(BytesWritable.class);

       // 5 设置输入输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

       // 6 提交job
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```



### 3.2 MapReduce工作流程

**1．流程示意图，如图**

![MapReduce3-17](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-17.png)

![MapReduce3-18](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-18.png)



**2．流程详解**

上面的流程是整个MapReduce最全工作流程，但是Shuffle过程只是从第7步开始到第16步结束，具体Shuffle过程详解，如下：

(1）MapTask收集我们的map()方法输出的kv对，放到内存缓冲区中

(2）从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件

(3）多个溢出文件会被合并成大的溢出文件

(4）在溢出过程及合并的过程中，都要调用Partitioner进行分区和针对key进行排序

(5）ReduceTask根据自己的分区号，去各个MapTask机器上取相应的结果分区数据

(6）ReduceTask会取到同一个分区的来自不同MapTask的结果文件，ReduceTask会将这些文件再进行合并（归并排序）

(7）合并成大文件后，Shuffle的过程也就结束了，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法）



**3．注意**

Shuffle中的缓冲区大小会影响到MapReduce程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快。

缓冲区的大小可以通过参数调整，参数：io.sort.mb默认100M。



**4．源码解析流程**

![MapReduce3-18-1](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-18-1.png)



### 3.3 Shuffle机制

**3.3.1 Shuffle机制**

Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle。Shuffle机制如图所示

![MapReduce3-19](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-19.png)



**3.3.2 Partition分区**

![MapReduce3-20](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-20.png)

自定义Partitioner图：

![MapReduce3-21](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-21.png)



**3.3.3 Partition分区案例实操**

**1．需求**

将统计结果按照手机归属地不同省份输出到不同文件中（分区）

（1）输入数据phone_data.txt

```
1	13736230513	192.196.100.1	www.cto15.com	2481	24681	200
2	13846544121	192.196.100.2			264	0	200
3 	13956435636	192.196.100.3			132	1512	200
4 	13966251146	192.168.100.1			240	0	404
5 	18271575951	192.168.100.2	www.cto15.com	1527	2106	200
6 	84188413	192.168.100.3	www.cto15.com	4116	1432	200
7 	13590439668	192.168.100.4			1116	954	200
8 	15910133277	192.168.100.5	www.hao123.com	3156	2936	200
9 	13729199489	192.168.100.6			240	0	200
10 	13630577991	192.168.100.7	www.shouhu.com	6960	690	200
11 	15043685818	192.168.100.8	www.baidu.com	3659	3538	200
12 	15959002129	192.168.100.9	www.cto15.com	1938	180	500
13 	13560439638	192.168.100.10			918	4938	200
14 	13470253144	192.168.100.11			180	180	200
15 	13682846555	192.168.100.12	www.qq.com	1938	2910	200
16 	13992314666	192.168.100.13	www.gaga.com	3008	3720	200
17 	13509468723	192.168.100.14	www.qinghua.com	7335	110349	404
18 	18390173782	192.168.100.15	www.sogou.com	9531	2412	200
19 	13975057813	192.168.100.16	www.baidu.com	11058	48243	200
20 	13768778790	192.168.100.17			120	120	200
21 	13568436656	192.168.100.18	www.alibaba.com	2481	24681	200
22 	13568436656	192.168.100.19			1116	954	200
```

（2）期望输出数据

手机号136、137、138、139开头都分别放到一个独立的4个文件中，其他开头的放到一个文件中。

**2．需求分析**

自定义Partitioner案例需求分析:

![MapReduce3-22](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-22.png)

**3．在案例2.4的基础上，增加一个分区类**

```java
package com.cto15.mapreduce.flowsum;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

public class ProvincePartitioner extends Partitioner<Text, FlowBean> {

	@Override
	public int getPartition(Text key, FlowBean value, int numPartitions) {

		// 1 获取电话号码的前三位
		String preNum = key.toString().substring(0, 3);
		
		int partition = 4;
		
		// 2 判断是哪个省
		if ("136".equals(preNum)) {
			partition = 0;
		}else if ("137".equals(preNum)) {
			partition = 1;
		}else if ("138".equals(preNum)) {
			partition = 2;
		}else if ("139".equals(preNum)) {
			partition = 3;
		}

		return partition;
	}
}
```

在驱动函数中增加自定义数据分区设置和ReduceTask设置

```java
package com.cto15.mapreduce.flowsum;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class FlowsumDriver {

	public static void main(String[] args) throws IllegalArgumentException, IOException, ClassNotFoundException, InterruptedException {

		// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
		args = new String[]{"e:/output1","e:/output2"};

		// 1 获取配置信息，或者job对象实例
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);

		// 2 指定本程序的jar包所在的本地路径
		job.setJarByClass(FlowsumDriver.class);

		// 3 指定本业务job要使用的mapper/Reducer业务类
		job.setMapperClass(FlowCountMapper.class);
		job.setReducerClass(FlowCountReducer.class);

		// 4 指定mapper输出数据的kv类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(FlowBean.class);

		// 5 指定最终输出的数据的kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(FlowBean.class);

		// 8 指定自定义数据分区
		job.setPartitionerClass(ProvincePartitioner.class);

		// 9 同时指定相应数量的reduce task
		job.setNumReduceTasks(5);
		
		// 6 指定job的输入原始文件所在目录
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```

![MapReduce3-23](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-23.png)



**3.3.4 WritableComparable排序**

![MapReduce3-24](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-24.png)

排序概述:

![MapReduce3-25](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-25.png)



**1．排序的分类**

![MapReduce3-26](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-26.png)



**2．自定义排序WritableComparable**

（1）原理分析

bean对象做为key传输，需要实现WritableComparable接口重写compareTo方法，就可以实现排序。

```java
@Override
public int compareTo(FlowBean o) {

	int result;
		
	// 按照总流量大小，倒序排列
	if (sumFlow > bean.getSumFlow()) {
		result = -1;
	}else if (sumFlow < bean.getSumFlow()) {
		result = 1;
	}else {
		result = 0;
	}

	return result;
}
```



**3.3.5 WritableComparable排序案例实操（全排序）**

**1．需求**

根据案例2.3产生的结果再次对总流量进行排序。

（1）输入数据 phone_data.txt

```
1	13736230513	192.196.100.1	www.cto15.com	2481	24681	200
2	13846544121	192.196.100.2			264	0	200
3 	13956435636	192.196.100.3			132	1512	200
4 	13966251146	192.168.100.1			240	0	404
5 	18271575951	192.168.100.2	www.cto15.com	1527	2106	200
6 	84188413	192.168.100.3	www.cto15.com	4116	1432	200
7 	13590439668	192.168.100.4			1116	954	200
8 	15910133277	192.168.100.5	www.hao123.com	3156	2936	200
9 	13729199489	192.168.100.6			240	0	200
10 	13630577991	192.168.100.7	www.shouhu.com	6960	690	200
11 	15043685818	192.168.100.8	www.baidu.com	3659	3538	200
12 	15959002129	192.168.100.9	www.cto15.com	1938	180	500
13 	13560439638	192.168.100.10			918	4938	200
14 	13470253144	192.168.100.11			180	180	200
15 	13682846555	192.168.100.12	www.qq.com	1938	2910	200
16 	13992314666	192.168.100.13	www.gaga.com	3008	3720	200
17 	13509468723	192.168.100.14	www.qinghua.com	7335	110349	404
18 	18390173782	192.168.100.15	www.sogou.com	9531	2412	200
19 	13975057813	192.168.100.16	www.baidu.com	11058	48243	200
20 	13768778790	192.168.100.17			120	120	200
21 	13568436656	192.168.100.18	www.alibaba.com	2481	24681	200
22 	13568436656	192.168.100.19			1116	954	200
```

第一次处理后的数据

part-r-00000

（2）期望输出数据

```
13509468723	7335	110349	117684
13736230513	2481	24681	27162
13956435636	132		1512	1644
13846544121	264		0		264
。。。 。。。
```



**2．需求分析**

WritableComparable排序案例分析

![MapReduce3-27](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-27.png)



**3．代码实现**

（1）FlowBean 

```java
package com.cto15.mapreduce.sort;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.WritableComparable;

public class FlowBean implements WritableComparable<FlowBean> {

	private long upFlow;
	private long downFlow;
	private long sumFlow;

	// 反序列化时，需要反射调用空参构造函数，所以必须有
	public FlowBean() {
		super();
	}

	public FlowBean(long upFlow, long downFlow) {
		super();
		this.upFlow = upFlow;
		this.downFlow = downFlow;
		this.sumFlow = upFlow + downFlow;
	}

	public void set(long upFlow, long downFlow) {
		this.upFlow = upFlow;
		this.downFlow = downFlow;
		this.sumFlow = upFlow + downFlow;
	}

	public long getSumFlow() {
		return sumFlow;
	}

	public void setSumFlow(long sumFlow) {
		this.sumFlow = sumFlow;
	}	

	public long getUpFlow() {
		return upFlow;
	}

	public void setUpFlow(long upFlow) {
		this.upFlow = upFlow;
	}

	public long getDownFlow() {
		return downFlow;
	}

	public void setDownFlow(long downFlow) {
		this.downFlow = downFlow;
	}

	/**
	 * 序列化方法
	 * @param out
	 * @throws IOException
	 */
	@Override
	public void write(DataOutput out) throws IOException {
		out.writeLong(upFlow);
		out.writeLong(downFlow);
		out.writeLong(sumFlow);
	}

	/**
	 * 反序列化方法 注意反序列化的顺序和序列化的顺序完全一致
	 * @param in
	 * @throws IOException
	 */
	@Override
	public void readFields(DataInput in) throws IOException {
		upFlow = in.readLong();
		downFlow = in.readLong();
		sumFlow = in.readLong();
	}

	@Override
	public String toString() {
		return upFlow + "\t" + downFlow + "\t" + sumFlow;
	}

	@Override
	public int compareTo(FlowBean bean) {
		
		int result;
		
		// 按照总流量大小，倒序排列
		if (sumFlow > bean.getSumFlow()) {
			result = -1;
		}else if (sumFlow < bean.getSumFlow()) {
			result = 1;
		}else {
			result = 0;
		}

		return result;
	}
}
```

（2）编写Mapper类

```java
package com.cto15.mapreduce.sort;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class FlowCountSortMapper extends Mapper<LongWritable, Text, FlowBean, Text>{

	FlowBean bean = new FlowBean();
	Text v = new Text();

	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {

		// 1 获取一行
		String line = value.toString();
		
		// 2 截取
		String[] fields = line.split("\t");
		
		// 3 封装对象
		String phoneNbr = fields[0];
		long upFlow = Long.parseLong(fields[1]);
		long downFlow = Long.parseLong(fields[2]);
		
		bean.set(upFlow, downFlow);
		v.set(phoneNbr);
		
		// 4 输出
		context.write(bean, v);
	}
}
```

（3）编写Reducer类

```java
package com.cto15.mapreduce.sort;
import java.io.IOException;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class FlowCountSortReducer extends Reducer<FlowBean, Text, Text, FlowBean>{

	@Override
	protected void reduce(FlowBean key, Iterable<Text> values, Context context)	throws IOException, InterruptedException {
		
		// 循环输出，避免总流量相同情况
		for (Text text : values) {
			context.write(text, key);
		}
	}
}
```

（4）编写Driver类

```java
package com.cto15.mapreduce.sort;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class FlowCountSortDriver {

	public static void main(String[] args) throws ClassNotFoundException, IOException, InterruptedException {

		// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
		args = new String[]{"e:/output1","e:/output2"};

		// 1 获取配置信息，或者job对象实例
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);

		// 2 指定本程序的jar包所在的本地路径
		job.setJarByClass(FlowCountSortDriver.class);

		// 3 指定本业务job要使用的mapper/Reducer业务类
		job.setMapperClass(FlowCountSortMapper.class);
		job.setReducerClass(FlowCountSortReducer.class);

		// 4 指定mapper输出数据的kv类型
		job.setMapOutputKeyClass(FlowBean.class);
		job.setMapOutputValueClass(Text.class);

		// 5 指定最终输出的数据的kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(FlowBean.class);

		// 6 指定job的输入原始文件所在目录
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		// 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```

**3.3.6 WritableComparable排序案例实操（区内排序）**

**1．需求**

要求每个省份手机号输出的文件中按照总流量内部排序。

**2．需求分析**

基于前一个需求，增加自定义分区类，分区按照省份手机号设置。

WritableComparable排序案例分析(区内排序):

![MapReduce3-28](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-28.png)

**3．案例实操WritableComparable排序案例分析(区内排序)**

（1）增加自定义分区类

```java
package com.cto15.mapreduce.sort;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

public class ProvincePartitioner extends Partitioner<FlowBean, Text> {

	@Override
	public int getPartition(FlowBean key, Text value, int numPartitions) {
		
		// 1 获取手机号码前三位
		String preNum = value.toString().substring(0, 3);
		
		int partition = 4;
		
		// 2 根据手机号归属地设置分区
		if ("136".equals(preNum)) {
			partition = 0;
		}else if ("137".equals(preNum)) {
			partition = 1;
		}else if ("138".equals(preNum)) {
			partition = 2;
		}else if ("139".equals(preNum)) {
			partition = 3;
		}

		return partition;
	}
}
```

（2）在驱动类中添加分区类

```java
// 加载自定义分区类
job.setPartitionerClass(ProvincePartitioner.class);

// 设置Reducetask个数
job.setNumReduceTasks(5);
```



**3.3.7 Combiner合并**

![MapReduce3-29](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-29.png)

（6）自定义Combiner实现步骤

（a）自定义一个Combiner继承Reducer，重写Reduce方法

```java
public class WordcountCombiner extends Reducer<Text, IntWritable, Text,IntWritable>{

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {

        // 1 汇总操作
		int count = 0;
		for(IntWritable v :values){
			count += v.get();
		}

        // 2 写出
		context.write(key, new IntWritable(count));
	}
}
```

（b）在Job驱动类中设置：

```
job.setCombinerClass(WordcountCombiner.class);
```



**3.3.8 Combiner合并案例实操**

**1．需求**

统计过程中对每一个MapTask的输出进行局部汇总，以减小网络传输量即采用Combiner功能。

（1）数据输入hello.txt

```
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
```

（2）期望输出数据

期望：Combine输入数据多，输出时经过合并，输出数据降低。

Combiner的合并案例对每一个MapTask的输出局部汇总如下：

![MapReduce3-30](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-30.png)



**3．案例实操-方案一**

（1）增加一个WordcountCombiner类继承Reducer

```java
package com.cto15.mr.combiner;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordcountCombiner extends Reducer<Text, IntWritable, Text, IntWritable>{

IntWritable v = new IntWritable();

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {

        // 1 汇总
		int sum = 0;

		for(IntWritable value :values){
			sum += value.get();
		}

		v.set(sum);

		// 2 写出
		context.write(key, v);
	}
}
```

（2）在WordcountDriver驱动类中指定Combiner

```
// 指定需要使用combiner，以及用哪个类作为combiner的逻辑
job.setCombinerClass(WordcountCombiner.class);
```



**4．案例实操-方案二**

（1）将WordcountReducer作为Combiner在WordcountDriver驱动类中指定

```
// 指定需要使用Combiner，以及用哪个类作为Combiner的逻辑
job.setCombinerClass(WordcountReducer.class);
```

运行程序，如图所示

未使用前：

![MapReduce3-31](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-31.png)

使用后：

![MapReduce3-32](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-32.png)



**3.3.9 GroupingComparator分组（辅助排序）**

对Reduce阶段的数据根据某一个或几个字段进行分组。

分组排序步骤：

（1）自定义类继承WritableComparator

（2）重写compare()方法

```java
@Override
public int compare(WritableComparable a, WritableComparable b) {
		// 比较的业务逻辑
		return result;
}
```

（3）创建一个构造将比较对象的类传给父类

```java
protected OrderGroupingComparator() {
		super(OrderBean.class, true);
}
```



**3.3.10 GroupingComparator分组案例实操**

**1．需求**

有如下订单数据

![MapReduce3-32-1](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-32-1.png)

现在需要求出每一个订单中最贵的商品。

（1）输入数据

GroupingComparator.txt

```
0000001	Pdt_01	222.8
0000002	Pdt_05	722.4
0000001	Pdt_02	33.8
0000003	Pdt_06	232.8
0000003	Pdt_02	33.8
0000002	Pdt_03	522.8
0000002	Pdt_04	122.4
```

（2）期望输出数据

```
1	222.8
2	722.4
3	232.8
```



**2．需求分析**

![MapReduce3-33](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-33.png)

**3．代码实现**

（1）定义订单信息OrderBean类

```java
package com.cto15.mapreduce.order;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.WritableComparable;

public class OrderBean implements WritableComparable<OrderBean> {

	private int order_id; // 订单id号
	private double price; // 价格

	public OrderBean() {
		super();
	}

	public OrderBean(int order_id, double price) {
		super();
		this.order_id = order_id;
		this.price = price;
	}

	@Override
	public void write(DataOutput out) throws IOException {
		out.writeInt(order_id);
		out.writeDouble(price);
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		order_id = in.readInt();
		price = in.readDouble();
	}

	@Override
	public String toString() {
		return order_id + "\t" + price;
	}

	public int getOrder_id() {
		return order_id;
	}

	public void setOrder_id(int order_id) {
		this.order_id = order_id;
	}

	public double getPrice() {
		return price;
	}

	public void setPrice(double price) {
		this.price = price;
	}

	// 二次排序
	@Override
	public int compareTo(OrderBean o) {

		int result;

		if (order_id > o.getOrder_id()) {
			result = 1;
		} else if (order_id < o.getOrder_id()) {
			result = -1;
		} else {
			// 价格倒序排序
			result = price > o.getPrice() ? -1 : 1;
		}

		return result;
	}
}
```

（2）编写OrderSortMapper类

```java
package com.cto15.mapreduce.order;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class OrderMapper extends Mapper<LongWritable, Text, OrderBean, NullWritable> {

	OrderBean k = new OrderBean();
	
	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		
		// 1 获取一行
		String line = value.toString();
		
		// 2 截取
		String[] fields = line.split("\t");
		
		// 3 封装对象
		k.setOrder_id(Integer.parseInt(fields[0]));
		k.setPrice(Double.parseDouble(fields[2]));
		
		// 4 写出
		context.write(k, NullWritable.get());
	}
}
```

（3）编写OrderSortGroupingComparator类

```java
package com.cto15.mapreduce.order;
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

public class OrderGroupingComparator extends WritableComparator {

	protected OrderGroupingComparator() {
		super(OrderBean.class, true);
	}

	@Override
	public int compare(WritableComparable a, WritableComparable b) {

		OrderBean aBean = (OrderBean) a;
		OrderBean bBean = (OrderBean) b;

		int result;
		if (aBean.getOrder_id() > bBean.getOrder_id()) {
			result = 1;
		} else if (aBean.getOrder_id() < bBean.getOrder_id()) {
			result = -1;
		} else {
			result = 0;
		}

		return result;
	}
}
```

（4）编写OrderSortReducer类

```java
package com.cto15.mapreduce.order;
import java.io.IOException;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

public class OrderReducer extends Reducer<OrderBean, NullWritable, OrderBean, NullWritable> {

	@Override
	protected void reduce(OrderBean key, Iterable<NullWritable> values, Context context)		throws IOException, InterruptedException {
		
		context.write(key, NullWritable.get());
	}
}
```

（5）编写OrderSortDriver类

```java
package com.cto15.mapreduce.order;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class OrderDriver {

	public static void main(String[] args) throws Exception, IOException {

// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
		args  = new String[]{"e:/input/inputorder" , "e:/output1"};

		// 1 获取配置信息
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);

		// 2 设置jar包加载路径
		job.setJarByClass(OrderDriver.class);

		// 3 加载map/reduce类
		job.setMapperClass(OrderMapper.class);
		job.setReducerClass(OrderReducer.class);

		// 4 设置map输出数据key和value类型
		job.setMapOutputKeyClass(OrderBean.class);
		job.setMapOutputValueClass(NullWritable.class);

		// 5 设置最终输出数据的key和value类型
		job.setOutputKeyClass(OrderBean.class);
		job.setOutputValueClass(NullWritable.class);

		// 6 设置输入数据和输出数据路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 8 设置reduce端的分组
	job.setGroupingComparatorClass(OrderGroupingComparator.class);

		// 7 提交
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```



### 3.4 MapTask工作机制

MapTask工作机制如图所示

![MapReduce3-34](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-34.png)

（1）Read阶段：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。

（2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。

（3）Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。

（4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

​    溢写阶段详情：

​    步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。

​    步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。

​    步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。

（5）Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

​    当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。

​    在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。

​    让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。



### 3.5 ReduceTask工作机制

**1．ReduceTask工作机制**

ReduceTask工作机制，如图

![MapReduce3-35](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-35.png)

（1）Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

​    （2）Merge阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。

​    （3）Sort阶段：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。

​    （4）Reduce阶段：reduce()函数将计算结果写到HDFS上。



**2．设置ReduceTask并行度（个数）**

ReduceTask的并行度同样影响整个Job的执行并发度和执行效率，但与MapTask的并发数由切片数决定不同，ReduceTask数量的决定是可以直接手动设置：

```
// 默认值是1，手动设置为4
job.setNumReduceTasks(4);
```



**3．实验：测试ReduceTask多少合适**

（1）实验环境：1个Master节点，16个Slave节点：CPU:8GHZ，内存: 2G

（2）实验结论：

表下 改变ReduceTask （数据量为1GB）

| MapTask =16 |      |      |      |      |      |      |      |      |      |      |
| ----------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| ReduceTask  | 1    | 5    | 10   | 15   | 16   | 20   | 25   | 30   | 45   | 60   |
| 总时间      | 892  | 146  | 110  | 92   | 88   | 100  | 128  | 101  | 145  | 104  |



**4．注意事项**

![MapReduce3-36](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-36.png)



### 3.6 OutputFormat数据输出

**3.6.1 OutputFormat接口实现类**

自定义OutputFormat

![MapReduce3-37](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-37.png)

**3.6.2 自定义OutputFormat**

![MapReduce3-38](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-38.png)

**3.6.3 自定义OutputFormat案例实操**

**1．需求**

过滤输入的log日志，包含cto15的网站输出到e:/cto15.log，不包含cto15的网站输出到e:/other.log。

（1）输入数据

log.txt

```
http://www.baidu.com
http://www.google.com
http://cn.bing.com
http://www.cto15.com
http://www.sohu.com
http://www.sina.com
http://www.sin2a.com
http://www.sin2desa.com
http://www.sindsafa.com
```

（2）期望输出数据

cto15.log

```
http://www.cto15.com
```

other.log

```
http://cn.bing.com
http://www.baidu.com
http://www.google.com
http://www.sin2a.com
http://www.sin2desa.com
http://www.sina.com
http://www.sindsafa.com
http://www.sohu.com
```

**2．需求分析**

自定义OutputFormat案例需求分析

![MapReduce3-39](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-39.png)



**3．案例实操**

（1）编写FilterMapper类

```java
package com.cto15.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class FilterMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {

		// 写出
		context.write(value, NullWritable.get());
	}
}
```

（2）编写FilterReducer类

```java
package com.cto15.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class FilterReducer extends Reducer<Text, NullWritable, Text, NullWritable> {

Text k = new Text();

	@Override
	protected void reduce(Text key, Iterable<NullWritable> values, Context context)		throws IOException, InterruptedException {

       // 1 获取一行
		String line = key.toString();

       // 2 拼接
		line = line + "\r\n";

       // 3 设置key
       k.set(line);

       // 4 输出
		context.write(k, NullWritable.get());
	}
}
```

（3）自定义一个OutputFormat类

```java
package com.cto15.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class FilterOutputFormat extends FileOutputFormat<Text, NullWritable>{

	@Override
	public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext job)			throws IOException, InterruptedException {

		// 创建一个RecordWriter
		return new FilterRecordWriter(job);
	}
}
```

（4）编写RecordWriter类

```java
package com.cto15.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;

public class FilterRecordWriter extends RecordWriter<Text, NullWritable> {

	FSDataOutputStream cto15Out = null;
	FSDataOutputStream otherOut = null;

	public FilterRecordWriter(TaskAttemptContext job) {

		// 1 获取文件系统
		FileSystem fs;

		try {
			fs = FileSystem.get(job.getConfiguration());

			// 2 创建输出文件路径
			Path cto15Path = new Path("e:/cto15.log");
			Path otherPath = new Path("e:/other.log");

			// 3 创建输出流
			cto15Out = fs.create(cto15Path);
			otherOut = fs.create(otherPath);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	@Override
	public void write(Text key, NullWritable value) throws IOException, InterruptedException {

		// 判断是否包含“cto15”输出到不同文件
		if (key.toString().contains("cto15")) {
			cto15Out.write(key.toString().getBytes());
		} else {
			otherOut.write(key.toString().getBytes());
		}
	}

	@Override
	public void close(TaskAttemptContext context) throws IOException, InterruptedException {

		// 关闭资源
IOUtils.closeStream(cto15Out);
		IOUtils.closeStream(otherOut);	}
}
```

（5）编写FilterDriver类

```java
package com.cto15.mapreduce.outputformat;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class FilterDriver {

	public static void main(String[] args) throws Exception {

// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
args = new String[] { "e:/input/inputoutputformat", "e:/output2" };

		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);

		job.setJarByClass(FilterDriver.class);
		job.setMapperClass(FilterMapper.class);
		job.setReducerClass(FilterReducer.class);

		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(NullWritable.class);
		
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		// 要将自定义的输出格式组件设置到job中
		job.setOutputFormatClass(FilterOutputFormat.class);

		FileInputFormat.setInputPaths(job, new Path(args[0]));

		// 虽然我们自定义了outputformat，但是因为我们的outputformat继承自fileoutputformat
		// 而fileoutputformat要输出一个_SUCCESS文件，所以，在这还得指定一个输出目录
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```



### 3.7 Join多种应用

**3.7.1 Reduce Join**

ReduceJoin工作原理

![MapReduce3-40](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-40.png)

**3.7.2 Reduce Join案例实操**

1．需求 order.txt

```
1001	01	1
1002	02	2
1003	03	3
1004	01	4
1005	02	5
1006	03	6
```

订单数据表t_order如下

| id   | pid  | amount |
| ---- | ---- | ------ |
| 1001 | 01   | 1      |
| 1002 | 02   | 2      |
| 1003 | 03   | 3      |
| 1004 | 01   | 4      |
| 1005 | 02   | 5      |
| 1006 | 03   | 6      |

------

pd.txt

```
01	小米
02	华为
03	格力
```

商品信息表t_product如下

| pid  | pname |
| ---- | ----- |
| 01   | 小米  |
| 02   | 华为  |
| 03   | 格力  |

------

将商品信息表中数据根据商品pid合并到订单数据表中:

最终数据形式

| id   | pname | amount |
| ---- | ----- | ------ |
| 1001 | 小米  | 1      |
| 1004 | 小米  | 4      |
| 1002 | 华为  | 2      |
| 1005 | 华为  | 5      |
| 1003 | 格力  | 3      |
| 1006 | 格力  | 6      |

2．需求分析

通过将关联条件作为Map输出的key，将两表满足Join条件的数据并携带数据所来源的文件信息，发往同一个ReduceTask，在Reduce中进行数据的串联， Reduce端表合并如图:

![MapReduce3-41](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-41.png)

3．代码实现

(1）创建商品和订合并后的Bean类

```java
package com.cto15.reducejoin;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class OrderBean implements WritableComparable<OrderBean> {
    private String id;
    private String pid;
    private int amount;
    private String pname;

    @Override
    public String toString() {
        return id + "\t" + pname + "\t" + amount;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getPid() {
        return pid;
    }

    public void setPid(String pid) {
        this.pid = pid;
    }

    public int getAmount() {
        return amount;
    }

    public void setAmount(int amount) {
        this.amount = amount;
    }

    public String getPname() {
        return pname;
    }

    public void setPname(String pname) {
        this.pname = pname;
    }

    //按照Pid分组，组内按照pname排序，有pname的在前
    @Override
    public int compareTo(OrderBean o) {
        int compare = this.pid.compareTo(o.pid);
        if (compare == 0) {
            return o.getPname().compareTo(this.getPname());
        } else {
            return compare;
        }
    }

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(id);
        out.writeUTF(pid);
        out.writeInt(amount);
        out.writeUTF(pname);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        id = in.readUTF();
        pid = in.readUTF();
        amount = in.readInt();
        pname = in.readUTF();
    }
}
```

(2）编写TableMapper类

```java
package com.cto15.reducejoin;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

import java.io.IOException;

public class OrderMapper extends Mapper<LongWritable, Text, OrderBean, NullWritable> {

    private String filename;

    private OrderBean order = new OrderBean();

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        
        //获取切片文件名
        FileSplit fs = (FileSplit) context.getInputSplit();
        filename = fs.getPath().getName();
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] fields = value.toString().split("\t");
        
        //对不同数据来源分开处理
        if ("order.txt".equals(filename)) {
            order.setId(fields[0]);
            order.setPid(fields[1]);
            order.setAmount(Integer.parseInt(fields[2]));
            order.setPname("");
        } else {
            order.setPid(fields[0]);
            order.setPname(fields[1]);
            order.setAmount(0);
            order.setId("");
        }

        context.write(order, NullWritable.get());
    }
}
```

(3）编写TableReducer类

```java
package com.cto15.reducejoin;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
import java.util.Iterator;

public class OrderReducer extends Reducer<OrderBean, NullWritable, OrderBean, NullWritable> {

    @Override
    protected void reduce(OrderBean key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        
        //第一条数据来自pd，之后全部来自order
        Iterator<NullWritable> iterator = values.iterator();
        
        //通过第一条数据获取pname
        iterator.next();
        String pname = key.getPname();
        
        //遍历剩下的数据，替换并写出
        while (iterator.hasNext()) {
            iterator.next();
            key.setPname(pname);
            context.write(key,NullWritable.get());
        }
    }
}
```

(4）编写TableDriver类

```java
package com.cto15.reducejoin;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class OrderDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Job job = Job.getInstance(new Configuration());
        job.setJarByClass(OrderDriver.class);

        job.setMapperClass(OrderMapper.class);
        job.setReducerClass(OrderReducer.class);
        job.setGroupingComparatorClass(OrderComparator.class);

        job.setMapOutputKeyClass(OrderBean.class);
        job.setMapOutputValueClass(NullWritable.class);

        job.setOutputKeyClass(OrderBean.class);
        job.setOutputValueClass(NullWritable.class);

        FileInputFormat.setInputPaths(job, new Path("d:\\input"));
        FileOutputFormat.setOutputPath(job, new Path("d:\\output"));

        boolean b = job.waitForCompletion(true);

        System.exit(b ? 0 : 1);

    }
}
```

4．测试

运行程序查看结果

```
1001	小米	1	
1001	小米	1	
1002	华为	2	
1002	华为	2	
1003	格力	3	
1003	格力	3	
```

5．总结

ReduceJoin缺点及解决方案

```
缺点：这种方式中，合并的操作是在Reduce阶段完成，Reduce端的处理压力太大，Map节点的运算负载很低，资源利用率不高，且在Reduce阶段极易产生数据倾斜
```

```
解决方案：Map端实现数据合并
```



**3.7.3 Map Join**

1．使用场景

Map Join适用于一张表十分小、一张表很大的场景。

2．优点

思考：在Reduce端处理过多的表，非常容易产生数据倾斜。怎么办？

在Map端缓存多张表，提前处理业务逻辑，这样增加Map端业务，减少Reduce端数据的压力，尽可能的减少数据倾斜。

3．具体办法：采用DistributedCache

​    （1）在Mapper的setup阶段，将文件读取到缓存集合中。

​    （2）在驱动函数中加载缓存。

​		// 缓存普通文件到Task运行节点。

job.addCacheFile(new URI("file://e:/cache/pd.txt"));



**3.7.4 Map Join案例实操**

1．需求

order.txt

```
1001	01	1
1002	02	2
1003	03	3
1004	01	4
1005	02	5
1006	03	6
```

订单数据表t_order

| id   | pid  | amount |
| ---- | ---- | ------ |
| 1001 | 01   | 1      |
| 1002 | 02   | 2      |
| 1003 | 03   | 3      |
| 1004 | 01   | 4      |
| 1005 | 02   | 5      |
| 1006 | 03   | 6      |

------

pd.txt

```
01	小米
02	华为
03	格力
```

商品信息表t_product

```
pid	pname
01	小米
02	华为
03	格力
```

将商品信息表中数据根据商品pid合并到订单数据表中。

------

最终数据形式

| id   | pname | amount |
| ---- | ----- | ------ |
| 1001 | 小米  | 1      |
| 1004 | 小米  | 4      |
| 1002 | 华为  | 2      |
| 1005 | 华为  | 5      |
| 1003 | 格力  | 3      |
| 1006 | 格力  | 6      |

**2．需求分析**

MapJoin适用于关联表中有小表的情形。

Map端表合并

![MapReduce3-43](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-43.png)

**3．实现代码**

（1）先在驱动模块中添加缓存文件

```java
package test;
import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class DistributedCacheDriver {

	public static void main(String[] args) throws Exception {
		
// 0 根据自己电脑路径重新配置
args = new String[]{"e:/input/inputtable2", "e:/output1"};

// 1 获取job信息
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);

		// 2 设置加载jar包路径
		job.setJarByClass(DistributedCacheDriver.class);

		// 3 关联map
		job.setMapperClass(DistributedCacheMapper.class);
		
// 4 设置最终输出数据类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		// 5 设置输入输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 6 加载缓存数据
		job.addCacheFile(new URI("file:///e:/input/inputcache/pd.txt"));
		
		// 7 Map端Join的逻辑不需要Reduce阶段，设置reduceTask数量为0
		job.setNumReduceTasks(0);

		// 8 提交
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
```

（2）读取缓存的文件数据

```java
package com.cto15.mapjoin;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URI;
import java.util.HashMap;
import java.util.Map;

public class MjMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

    //pd表在内存中的缓存
    private Map<String, String> pMap = new HashMap<>();

    private Text line = new Text();

    //任务开始前将pd数据缓存进PMap
    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        
        //从缓存文件中找到pd.txt
        URI[] cacheFiles = context.getCacheFiles();
        Path path = new Path(cacheFiles[0]);

        //获取文件系统并开流
        FileSystem fileSystem = FileSystem.get(context.getConfiguration());
        FSDataInputStream fsDataInputStream = fileSystem.open(path);

        //通过包装流转换为reader
        BufferedReader bufferedReader = new BufferedReader(
                new InputStreamReader(fsDataInputStream, "utf-8"));

        //逐行读取，按行处理
        String line;
        while (StringUtils.isNotEmpty(line = bufferedReader.readLine())) {
            String[] fields = line.split("\t");
            pMap.put(fields[0], fields[1]);
        }

        //关流
        IOUtils.closeStream(bufferedReader);

    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] fields = value.toString().split("\t");

        String pname = pMap.get(fields[1]);

        line.set(fields[0] + "\t" + pname + "\t" + fields[2]);

        context.write(line, NullWritable.get());

    }
}
```



### 3.8 计数器应用

![MapReduce3-44](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-44.png)



### 3.9 数据清洗（ETL）

在运行核心业务MapReduce程序之前，往往要先对数据进行清洗，清理掉不符合用户要求的数据。清理的过程往往只需要运行Mapper程序，不需要运行Reduce程序。

**3.9.1 数据清洗案例实操-简单解析版**

**1．需求**

去除日志中字段个数小于等于11的日志

（1）输入数据

web.log

```
194.237.142.21 - - [18/Sep/2013:06:49:18 +0000] "GET /wp-content/uploads/2013/07/rstudio-git3.png HTTP/1.1" 304 0 "-" "Mozilla/4.0 (compatible;)"
183.49.46.228 - - [18/Sep/2013:06:49:23 +0000] "-" 400 0 "-" "-"
163.177.71.12 - - [18/Sep/2013:06:49:33 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
163.177.71.12 - - [18/Sep/2013:06:49:36 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
101.226.68.137 - - [18/Sep/2013:06:49:42 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
101.226.68.137 - - [18/Sep/2013:06:49:45 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
60.208.6.156 - - [18/Sep/2013:06:49:48 +0000] "GET /wp-content/uploads/2013/07/rcassandra.png HTTP/1.0" 200 185524 "http://cos.name/category/software/packages/" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.66 Safari/537.36"
222.68.172.190 - - [18/Sep/2013:06:49:57 +0000] "GET /images/my.jpg HTTP/1.1" 200 19939 "http://www.angularjs.cn/A00n" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.66 Safari/537.36"
222.68.172.190 - - [18/Sep/2013:06:50:08 +0000] "-" 400 0 "-" "-"
183.195.232.138 - - [18/Sep/2013:06:50:16 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
183.195.232.138 - - [18/Sep/2013:06:50:16 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
66.249.66.84 - - [18/Sep/2013:06:50:28 +0000] "GET /page/6/ HTTP/1.1" 200 27777 "-" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
221.130.41.168 - - [18/Sep/2013:06:50:37 +0000] "GET /feed/ HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.66 Safari/537.36"
157.55.35.40 - - [18/Sep/2013:06:51:13 +0000] "GET /robots.txt HTTP/1.1" 200 150 "-" "Mozilla/5.0 (compatible; bingbot/2.0; +http://www.bing.com/bingbot.htm)"
50.116.27.194 - - [18/Sep/2013:06:51:35 +0000] "POST /wp-cron.php?doing_wp_cron=1379487095.2510800361633300781250 HTTP/1.0" 200 0 "-" "WordPress/3.6; http://blog.fens.me"
58.215.204.118 - - [18/Sep/2013:06:51:35 +0000] "GET /nodejs-socketio-chat/ HTTP/1.1" 200 10818 "http://www.google.com/url?sa=t&rct=j&q=nodejs%20%E5%BC%82%E6%AD%A5%E5%B9%BF%E6%92%AD&source=web&cd=1&cad=rja&ved=0CCgQFjAA&url=%68%74%74%70%3a%2f%2f%62%6c%6f%67%2e%66%65%6e%73%2e%6d%65%2f%6e%6f%64%65%6a%73%2d%73%6f%63%6b%65%74%69%6f%2d%63%68%61%74%2f&ei=rko5UrylAefOiAe7_IGQBw&usg=AFQjCNG6YWoZsJ_bSj8kTnMHcH51hYQkAA&bvm=bv.52288139,d.aGc" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:36 +0000] "GET /wp-includes/js/jquery/jquery-migrate.min.js?ver=1.2.1 HTTP/1.1" 304 0 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:35 +0000] "GET /wp-includes/js/jquery/jquery.js?ver=1.10.2 HTTP/1.1" 304 0 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:36 +0000] "GET /wp-includes/js/comment-reply.min.js?ver=3.6 HTTP/1.1" 304 0 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:36 +0000] "GET /wp-content/uploads/2013/08/chat.png HTTP/1.1" 200 48968 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:36 +0000] "GET /wp-content/uploads/2013/08/chat2.png HTTP/1.1" 200 59852 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:37 +0000] "GET /wp-content/uploads/2013/08/socketio.png HTTP/1.1" 200 80493 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.248.178.212 - - [18/Sep/2013:06:51:37 +0000] "GET /nodejs-grunt-intro/ HTTP/1.1" 200 51770 "http://blog.fens.me/series-nodejs/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/jquery/jquery-migrate.min.js?ver=1.2.1 HTTP/1.1" 200 7200 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/comment-reply.min.js?ver=3.6 HTTP/1.1" 200 786 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/jquery/jquery.js?ver=1.10.2 HTTP/1.1" 200 45307 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/jquery/jquery.js?ver=1.10.2 HTTP/1.1" 200 93128 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/comment-reply.min.js?ver=3.6 HTTP/1.1" 200 786 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.215.204.118 - - [18/Sep/2013:06:51:41 +0000] "-" 400 0 "-" "-"
58.215.204.118 - - [18/Sep/2013:06:51:41 +0000] "-" 400 0 "-" "-"
```

（2）期望输出数据

每行字段长度都大于11。

**2．需求分析**

需要在Map阶段对输入的数据根据规则进行过滤清洗。

**3．实现代码**

（1）编写LogMapper类

```java
package com.cto15.mapreduce.weblog;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class LogMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
	
	Text k = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		
		// 1 获取1行数据
		String line = value.toString();
		
		// 2 解析日志
		boolean result = parseLog(line,context);
		
		// 3 日志不合法退出
		if (!result) {
			return;
		}
		
		// 4 设置key
		k.set(line);
		
		// 5 写出数据
		context.write(k, NullWritable.get());
	}

	// 2 解析日志
	private boolean parseLog(String line, Context context) {

		// 1 截取
		String[] fields = line.split(" ");
		
		// 2 日志长度大于11的为合法
		if (fields.length > 11) {

			// 系统计数器
			context.getCounter("map", "true").increment(1);
			return true;
		}else {
			context.getCounter("map", "false").increment(1);
			return false;
		}
	}
}
```

（2）编写LogDriver类

```java
package com.cto15.mapreduce.weblog;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class LogDriver {

	public static void main(String[] args) throws Exception {

// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
        args = new String[] { "e:/input/inputlog", "e:/output1" };

		// 1 获取job信息
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);

		// 2 加载jar包
		job.setJarByClass(LogDriver.class);

		// 3 关联map
		job.setMapperClass(LogMapper.class);

		// 4 设置最终输出类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		// 设置reducetask个数为0
		job.setNumReduceTasks(0);

		// 5 设置输入和输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 6 提交
		job.waitForCompletion(true);
	}
}
```

**3.9.2 数据清洗案例实操-复杂解析版**

**1．需求**

对Web访问日志中的各字段识别切分，去除日志中不合法的记录。根据清洗规则，输出过滤后的数据。

（1）输入数据

```
194.237.142.21 - - [18/Sep/2013:06:49:18 +0000] "GET /wp-content/uploads/2013/07/rstudio-git3.png HTTP/1.1" 304 0 "-" "Mozilla/4.0 (compatible;)"
183.49.46.228 - - [18/Sep/2013:06:49:23 +0000] "-" 400 0 "-" "-"
163.177.71.12 - - [18/Sep/2013:06:49:33 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
163.177.71.12 - - [18/Sep/2013:06:49:36 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
101.226.68.137 - - [18/Sep/2013:06:49:42 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
101.226.68.137 - - [18/Sep/2013:06:49:45 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
60.208.6.156 - - [18/Sep/2013:06:49:48 +0000] "GET /wp-content/uploads/2013/07/rcassandra.png HTTP/1.0" 200 185524 "http://cos.name/category/software/packages/" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.66 Safari/537.36"
222.68.172.190 - - [18/Sep/2013:06:49:57 +0000] "GET /images/my.jpg HTTP/1.1" 200 19939 "http://www.angularjs.cn/A00n" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.66 Safari/537.36"
222.68.172.190 - - [18/Sep/2013:06:50:08 +0000] "-" 400 0 "-" "-"
183.195.232.138 - - [18/Sep/2013:06:50:16 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
183.195.232.138 - - [18/Sep/2013:06:50:16 +0000] "HEAD / HTTP/1.1" 200 20 "-" "DNSPod-Monitor/1.0"
66.249.66.84 - - [18/Sep/2013:06:50:28 +0000] "GET /page/6/ HTTP/1.1" 200 27777 "-" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
221.130.41.168 - - [18/Sep/2013:06:50:37 +0000] "GET /feed/ HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.66 Safari/537.36"
157.55.35.40 - - [18/Sep/2013:06:51:13 +0000] "GET /robots.txt HTTP/1.1" 200 150 "-" "Mozilla/5.0 (compatible; bingbot/2.0; +http://www.bing.com/bingbot.htm)"
50.116.27.194 - - [18/Sep/2013:06:51:35 +0000] "POST /wp-cron.php?doing_wp_cron=1379487095.2510800361633300781250 HTTP/1.0" 200 0 "-" "WordPress/3.6; http://blog.fens.me"
58.215.204.118 - - [18/Sep/2013:06:51:35 +0000] "GET /nodejs-socketio-chat/ HTTP/1.1" 200 10818 "http://www.google.com/url?sa=t&rct=j&q=nodejs%20%E5%BC%82%E6%AD%A5%E5%B9%BF%E6%92%AD&source=web&cd=1&cad=rja&ved=0CCgQFjAA&url=%68%74%74%70%3a%2f%2f%62%6c%6f%67%2e%66%65%6e%73%2e%6d%65%2f%6e%6f%64%65%6a%73%2d%73%6f%63%6b%65%74%69%6f%2d%63%68%61%74%2f&ei=rko5UrylAefOiAe7_IGQBw&usg=AFQjCNG6YWoZsJ_bSj8kTnMHcH51hYQkAA&bvm=bv.52288139,d.aGc" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:36 +0000] "GET /wp-includes/js/jquery/jquery-migrate.min.js?ver=1.2.1 HTTP/1.1" 304 0 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:35 +0000] "GET /wp-includes/js/jquery/jquery.js?ver=1.10.2 HTTP/1.1" 304 0 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:36 +0000] "GET /wp-includes/js/comment-reply.min.js?ver=3.6 HTTP/1.1" 304 0 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:36 +0000] "GET /wp-content/uploads/2013/08/chat.png HTTP/1.1" 200 48968 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:36 +0000] "GET /wp-content/uploads/2013/08/chat2.png HTTP/1.1" 200 59852 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.215.204.118 - - [18/Sep/2013:06:51:37 +0000] "GET /wp-content/uploads/2013/08/socketio.png HTTP/1.1" 200 80493 "http://blog.fens.me/nodejs-socketio-chat/" "Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0"
58.248.178.212 - - [18/Sep/2013:06:51:37 +0000] "GET /nodejs-grunt-intro/ HTTP/1.1" 200 51770 "http://blog.fens.me/series-nodejs/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/jquery/jquery-migrate.min.js?ver=1.2.1 HTTP/1.1" 200 7200 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/comment-reply.min.js?ver=3.6 HTTP/1.1" 200 786 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/jquery/jquery.js?ver=1.10.2 HTTP/1.1" 200 45307 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/jquery/jquery.js?ver=1.10.2 HTTP/1.1" 200 93128 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.248.178.212 - - [18/Sep/2013:06:51:40 +0000] "GET /wp-includes/js/comment-reply.min.js?ver=3.6 HTTP/1.1" 200 786 "http://blog.fens.me/nodejs-grunt-intro/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; MDDR; InfoPath.2; .NET4.0C)"
58.215.204.118 - - [18/Sep/2013:06:51:41 +0000] "-" 400 0 "-" "-"
58.215.204.118 - - [18/Sep/2013:06:51:41 +0000] "-" 400 0 "-" "-"
58.215.204.118 - - [18/Sep/2013:06:51:41 +0000] "-" 400 0 "-" "-"
```

（2）期望输出数据

都是合法的数据

**2．实现代码**

（1）定义一个bean，用来记录日志数据中的各数据字段

```java
package com.cto15.mapreduce.log;

public class LogBean {
	private String remote_addr;// 记录客户端的ip地址
	private String remote_user;// 记录客户端用户名称,忽略属性"-"
	private String time_local;// 记录访问时间与时区
	private String request;// 记录请求的url与http协议
	private String status;// 记录请求状态；成功是200
	private String body_bytes_sent;// 记录发送给客户端文件主体内容大小
	private String http_referer;// 用来记录从那个页面链接访问过来的
	private String http_user_agent;// 记录客户浏览器的相关信息

	private boolean valid = true;// 判断数据是否合法

	public String getRemote_addr() {
		return remote_addr;
	}

	public void setRemote_addr(String remote_addr) {
		this.remote_addr = remote_addr;
	}

	public String getRemote_user() {
		return remote_user;
	}

	public void setRemote_user(String remote_user) {
		this.remote_user = remote_user;
	}

	public String getTime_local() {
		return time_local;
	}

	public void setTime_local(String time_local) {
		this.time_local = time_local;
	}

	public String getRequest() {
		return request;
	}

	public void setRequest(String request) {
		this.request = request;
	}

	public String getStatus() {
		return status;
	}

	public void setStatus(String status) {
		this.status = status;
	}

	public String getBody_bytes_sent() {
		return body_bytes_sent;
	}

	public void setBody_bytes_sent(String body_bytes_sent) {
		this.body_bytes_sent = body_bytes_sent;
	}

	public String getHttp_referer() {
		return http_referer;
	}

	public void setHttp_referer(String http_referer) {
		this.http_referer = http_referer;
	}

	public String getHttp_user_agent() {
		return http_user_agent;
	}

	public void setHttp_user_agent(String http_user_agent) {
		this.http_user_agent = http_user_agent;
	}

	public boolean isValid() {
		return valid;
	}

	public void setValid(boolean valid) {
		this.valid = valid;
	}

	@Override
	public String toString() {

		StringBuilder sb = new StringBuilder();
		sb.append(this.valid);
		sb.append("\001").append(this.remote_addr);
		sb.append("\001").append(this.remote_user);
		sb.append("\001").append(this.time_local);
		sb.append("\001").append(this.request);
		sb.append("\001").append(this.status);
		sb.append("\001").append(this.body_bytes_sent);
		sb.append("\001").append(this.http_referer);
		sb.append("\001").append(this.http_user_agent);
		
		return sb.toString();
	}
}
```

（2）编写LogMapper类

```java
package com.cto15.mapreduce.log;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class LogMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
	Text k = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {

		// 1 获取1行
		String line = value.toString();
		
		// 2 解析日志是否合法
		LogBean bean = parseLog(line);
		
		if (!bean.isValid()) {
			return;
		}
		
		k.set(bean.toString());
		
		// 3 输出
		context.write(k, NullWritable.get());
	}

	// 解析日志
	private LogBean parseLog(String line) {

		LogBean logBean = new LogBean();
		
		// 1 截取
		String[] fields = line.split(" ");
		
		if (fields.length > 11) {

			// 2封装数据
			logBean.setRemote_addr(fields[0]);
			logBean.setRemote_user(fields[1]);
			logBean.setTime_local(fields[3].substring(1));
			logBean.setRequest(fields[6]);
			logBean.setStatus(fields[8]);
			logBean.setBody_bytes_sent(fields[9]);
			logBean.setHttp_referer(fields[10]);
			
			if (fields.length > 12) {
				logBean.setHttp_user_agent(fields[11] + " "+ fields[12]);
			}else {
				logBean.setHttp_user_agent(fields[11]);
			}
			
			// 大于400，HTTP错误
			if (Integer.parseInt(logBean.getStatus()) >= 400) {
				logBean.setValid(false);
			}
		}else {
			logBean.setValid(false);
		}
		
		return logBean;
	}
}
```

（3）编写LogDriver类

```java
package com.cto15.mapreduce.log;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class LogDriver {
	public static void main(String[] args) throws Exception {
		
// 1 获取job信息
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);

		// 2 加载jar包
		job.setJarByClass(LogDriver.class);

		// 3 关联map
		job.setMapperClass(LogMapper.class);

		// 4 设置最终输出类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		// 5 设置输入和输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 6 提交
		job.waitForCompletion(true);
	}
}
```



### 3.10 MapReduce开发总结

在编写MapReduce程序时，需要考虑如下几个方面：

![MapReduce3-45](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-45.png)

![MapReduce3-46](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-46.png)

![MapReduce3-47](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-47.png)

![MapReduce3-48](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-48.png)

![MapReduce3-49](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/MapReduce3-49.png)



## 参考资料

1. [Apache Hadoop 3.1.3 > MapReduce ](https://hadoop.apache.org/docs/stable/)
2. 尚硅谷大数据文档
