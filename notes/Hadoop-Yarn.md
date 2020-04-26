# Yarn--资源调度器

<nav>
<a href="#一、Yarn基本架构">一、Yarn基本架构</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#1.1 Yarn简介">1.1 Yarn简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#1.2 Yarn基本架构">1.2 Yarn基本架构</a><br/>
<a href="#二、Yarn工作机制">二、Yarn工作机制</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#2.1 Yarn运行机制">2.1 Yarn运行机制</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#2.2 工作机制详解">2.2 工作机制详解（Writable）</a><br/>
<a href="#三、作业提交全过程">三、作业提交全过程</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#3.1 作业提交过程之YARN">3.1 作业提交过程之YARN</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#3.2 作业提交过程之MapReduce">3.2 作业提交过程之MapReduce</a><br/>
<a href="#四、资源调度器">四、资源调度器</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#4.1 先进先出调度器（FIFO）">4.1 先进先出调度器（FIFO）</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#4.2 容量调度器（Capacity Scheduler）">4.2 容量调度器（Capacity Scheduler）</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#4.3 公平调度器（Fair Scheduler）">4.3 公平调度器（Fair Scheduler）</a><br/>
<a href="#五、容量调度器多队列提交案例">五、容量调度器多队列提交案例</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#5.1 需求">5.1 需求</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#5.2 配置多队列的容量调度器">5.2 配置多队列的容量调度器</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#5.3 向Hive队列提交任务">5.3 向Hive队列提交任务</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#5.4 任务的推测执行">5.4 任务的推测执行</a><br/>
</nav>




## 一、Yarn基本架构

### 1.1 Yarn简介

Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。



### 1.2 Yarn基本架构

YARN主要由ResourceManager、NodeManager、ApplicationMaster和Container等组件构成，如图

![Yarn5-1](E:\BigData-Hadoop\picture\Yarn5-1.png)



## 二、Yarn工作机制

### 2.1 Yarn运行机制

![Yarn5-2](E:\BigData-Hadoop\picture\Yarn5-2.png)

### 2.2 工作机制详解

（1）MR程序提交到客户端所在的节点。

（2）YarnRunner向ResourceManager申请一个Application。

（3）RM将该应用程序的资源路径返回给YarnRunner。

（4）该程序将运行所需资源提交到HDFS上。

（5）程序资源提交完毕后，申请运行mrAppMaster。

（6）RM将用户的请求初始化成一个Task。

（7）其中一个NodeManager领取到Task任务。

（8）该NodeManager创建容器Container，并产生MRAppmaster。

（9）Container从HDFS上拷贝资源到本地。

（10）MRAppmaster向RM 申请运行MapTask资源。

（11）RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

（12）MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

（13）MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

（14）ReduceTask向MapTask获取相应分区的数据。

（15）程序运行完毕后，MR会向RM申请注销自己。



## 三、作业提交全过程

### 3.1 作业提交过程之YARN

![Yarn5-3](E:\BigData-Hadoop\picture\Yarn5-3.png)

作业提交全过程详解

**（1）作业提交**

第1步：Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。

第2步：Client向RM申请一个作业id。

第3步：RM给Client返回该job资源的提交路径和作业id。

第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。

第5步：Client提交完资源后，向RM申请运行MrAppMaster。

**（2）作业初始化**

第6步：当RM收到Client的请求后，将该job添加到容量调度器中。

第7步：某一个空闲的NM领取到该Job。

第8步：该NM创建Container，并产生MRAppmaster。

第9步：下载Client提交的资源到本地。

**（3）任务分配**

第10步：MrAppMaster向RM申请运行多个MapTask任务资源。

第11步：RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

**（4）任务运行**

第12步：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

第14步：ReduceTask向MapTask获取相应分区的数据。

第15步：程序运行完毕后，MR会向RM申请注销自己。

**（5）进度和状态更新**

YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。

**（6）作业完成**

除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。



### 3.2 作业提交过程之MapReduce

![Yarn5-4](E:\BigData-Hadoop\picture\Yarn5-4.png)



## 四、资源调度器

目前，Hadoop作业调度器主要有三种：FIFO、Capacity Scheduler和Fair Scheduler。Hadoop3.1.3默认的资源调度器是Capacity Scheduler。

具体设置详见：yarn-default.xml文件

```shell
<property>
    <description>The class to use as the resource scheduler.</description>
    <name>yarn.resourcemanager.scheduler.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>
```

### 4.1 先进先出调度器（FIFO）

![Yarn5-5](E:\BigData-Hadoop\picture\Yarn5-5.png)

### 4.2 容量调度器（Capacity Scheduler）

![Yarn5-6](E:\BigData-Hadoop\picture\Yarn5-6.png)

### 4.3 公平调度器（Fair Scheduler）

![Yarn5-7](E:\BigData-Hadoop\picture\Yarn5-7.png)

![Yarn5-8](E:\BigData-Hadoop\picture\Yarn5-8.png)

## 五、容量调度器多队列提交案例

### 5.1 需求

Yarn默认的容量调度器是一条单队列的调度器，在实际使用中会出现单个任务阻塞整个队列的情况。同时，随着业务的增长，公司需要分业务限制集群使用率。这就需要我们按照业务种类配置多条任务队列。

### 5.2 配置多队列的容量调度器

默认Yarn的配置下，容量调度器只有一条Default队列。在capacity-scheduler.xml中可以配置多条队列，并降低default队列资源占比：

```
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,hive</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
  </property>
<property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>40</value>
  </property>
```

同时为新加队列添加必要属性：

```
<!--队列目标资源百分比，所有队列相加必须等于100-->
<property>
    <name>yarn.scheduler.capacity.root.hive.capacity</name>
    <value>60</value>
  </property>
<!--队列最大资源百分比-->
  <property>
    <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
    <value>100</value>
  </property>
<!—单用户可用队列资源占比-->
  <property>
    <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
    <value>1</value>
  </property>
<!--队列状态（RUNNING或STOPPING）-->
  <property>
    <name>yarn.scheduler.capacity.root.hive.state</name>
    <value>RUNNING</value>
  </property>
<!—队列允许哪些用户提交-->
  <property>
    <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
    <value>*</value>
  </property>
<!—队列允许哪些用户管理-->
<property>
    <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
    <value>*</value>
  </property>
```

在配置完成后，重启Yarn，就可以看到两条队列：

![Yarn5-9](E:\BigData-Hadoop\picture\Yarn5-9.png)

### 5.3 向Hive队列提交任务

默认的任务提交都是提交到default队列的。如果希望向其他队列提交任务，需要在Driver中声明：

```
public class WcDrvier {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = new Configuration();

        configuration.set("mapred.job.queue.name", "hive");

        //1. 获取一个Job实例
        Job job = Job.getInstance(configuration);

        //2. 设置类路径
        job.setJarByClass(WcDrvier.class);

        //3. 设置Mapper和Reducer
        job.setMapperClass(WcMapper.class);
        job.setReducerClass(WcReducer.class);

        //4. 设置Mapper和Reducer的输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        job.setCombinerClass(WcReducer.class);

        //5. 设置输入输出文件
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        //6. 提交Job
        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);
    }
}
```

这样，这个任务在集群提交时，就会提交到hive队列：

![Yarn5-10](E:\BigData-Hadoop\picture\Yarn5-10.png)



### 5.4 任务的推测执行

**1．作业完成时间取决于最慢的任务完成时间**

一个作业由若干个Map任务和Reduce任务构成。因硬件老化、软件Bug等，某些任务可能运行非常慢。

思考：系统中有99%的Map任务都完成了，只有少数几个Map老是进度很慢，完不成，怎么办？

**2．推测执行机制**

发现拖后腿的任务，比如某个任务运行速度远慢于任务平均速度。为拖后腿任务启动一个备份任务，同时运行。谁先运行完，则采用谁的结果。

**3．执行推测任务的前提条件**

1）每个Task只能有一个备份任务

（2）当前Job已完成的Task必须不小于0.05（5%）

（3）开启推测执行参数设置。mapred-site.xml文件中默认是打开的。

```
<property>
  	<name>mapreduce.map.speculative</name>
  	<value>true</value>
  	<description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
</property>

<property>
  	<name>mapreduce.reduce.speculative</name>
  	<value>true</value>
  	<description>If true, then multiple instances of some reduce tasks may be executed in parallel.</description>
</property>
```

**4．不能启用推测执行机制情况**

（1）任务间存在严重的负载倾斜；

（2）特殊任务，比如任务向数据库中写数据。

**5．算法原理，如图**

![Yarn5-11](E:\BigData-Hadoop\picture\Yarn5-11.png)