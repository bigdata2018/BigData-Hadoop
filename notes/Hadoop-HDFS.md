# HDFS--分布式文件存储系统

<nav>
<a href="#一、HDFS简介">一、HDFS简介</a><br/>
<a href="#二、HDFS 设计原理">二、HDFS 设计原理</a><br/>
    <a href="#三、HDFS的Shell操作（重点）">三、HDFS的Shell操作（重点）</a><br/>
    <a href="#四、HDFS客户端操作（重点）">四、HDFS客户端操作（重点）</a><br/>
    <a href="#五、HDFS的API操作">五、HDFS的API操作</a><br/>
    <a href="#六、HDFS的数据流（重点）">六、HDFS的数据流（重点）</a><br/>
    <a href="#七、NameNode和SecondaryNameNode（重点）">七、NameNode和SecondaryNameNode（重点）</a><br/>
    <a href="#八、DataNode(重点)">八、DataNode(重点)</a><br/>
    <a href="#九、小文件存档">九、小文件存档</a><br/>
</nav>



## 一、HDFS简介

1.**HDFS （Hadoop Distributed File System）**,它是一个文件系统，用于存储文件，通过目录树来定位文件；其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。

2.HDFS适合一次写入，多次读出的场景，且不支持文件的修改，适合用来做大量数据存储、分析。

3.HDFS优点：它具有高容错性；适合处理大数据；可构建在廉价机器上

4.HDFS不足：不支持并发写入、文件随机修改；不适合低延迟数据访问；无法高效对大量小文件进行存储



## 二、HDFS 设计原理

### 2.1 HDFS 架构

![HDFS1-4](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS1-4.png)



![HDFS1-5](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS1-5.png)



### 2.2 文件系统命名空间

HDFS 的 ` 文件系统命名空间 ` 的层次结构与大多数文件系统类似 (如 Linux)， 支持目录和文件的创建、移动、删除和重命名等操作，支持配置用户和访问权限，但不支持硬链接和软连接。`NameNode` 负责维护文件系统名称空间，记录对名称空间或其属性的任何更改。



### 2.3 数据复制

由于 Hadoop 被设计运行在廉价的机器上，这意味着硬件是不可靠的，为了保证容错性，HDFS 提供了数据复制机制。HDFS 将每一个文件存储为一系列**块**，每个块由多个副本来保证容错，块的大小和复制因子可以自行配置（默认情况下，块大小是 128M，默认复制因子是 3）。



### 2.4 HDFS文件块大小（重点）

![HDFS1-6](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS1-6.png)

**为什么块的大小不能设太大，也不能设太小？**

1.块设置太小，会增加寻址时间，程序一直在找块的开始位置

2.块设置太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。导致程序在处理这个块数据时，会非常慢。

**总结：HDFS块的大小设置主要取决于磁盘传输速率**



## 三、HDFS的Shell操作（重点）

### 3.1 基本语法

方式一：bin/hadoop fs 具体命令 

方式二：bin/hdfs dfs 具体命令



### 3.2 常用命令

1.启动Hadoop集群（方便后续的测试）

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ sbin/start-dfs.sh
[nogc@hadoop103 hadoop-3.1.3]$ sbin/start-yarn.sh
~~~

2.-help：输出这个命令参数

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -help rm
~~~

3.-ls: 显示目录信息

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -ls /
~~~

4.-mkdir：在HDFS上创建目录

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -mkdir -p /sanguo/shuguo
~~~

5.-moveFromLocal：从本地剪切粘贴到HDFS

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ touch kongming.txt
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs  -moveFromLocal  ./kongming.txt  /sanguo/shuguo
~~~

6.-appendToFile：追加一个文件到已经存在的文件末尾

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ touch liubei.txt
[nogc@hadoop102 hadoop-3.1.3]$ vi liubei.txt
输入
san gu mao lu
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -appendToFile liubei.txt /sanguo/shuguo/kongming.txt
~~~

7.-cat：显示文件内容

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -cat /sanguo/shuguo/kongming.txt
~~~

8.-chgrp 、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs  -chmod  666  /sanguo/shuguo/kongming.txt
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs  -chown  nogc:nogc   /sanguo/shuguo/kongming.txt
~~~

9.-copyFromLocal：从本地文件系统中拷贝文件到HDFS路径去

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -copyFromLocal README.txt /
~~~

10.-copyToLocal：从HDFS拷贝到本地

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -copyToLocal /sanguo/shuguo/kongming.txt ./
~~~

11.-cp ：从HDFS的一个路径拷贝到HDFS的另一个路径

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -cp /sanguo/shuguo/kongming.txt /zhuge.txt
~~~

12.-mv：在HDFS目录中移动文件

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -mv /zhuge.txt /sanguo/shuguo/
~~~

13.-get：等同于copyToLocal，就是从HDFS下载文件到本地

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -get /sanguo/shuguo/kongming.txt ./
~~~

14.-getmerge：合并下载多个文件，比如HDFS的目录 /user/nogc/test下有多个文件:log.1, log.2,log.3,...

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -getmerge /user/nogc/test/* ./zaiyiqi.txt
~~~

15.-put：等同于copyFromLocal

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -put ./zaiyiqi.txt /user/nogc/test/
~~~

16.-tail：显示一个文件的末尾

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -tail /sanguo/shuguo/kongming.txt
~~~

17.-rm：删除文件或文件夹

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -rm /user/nogc/test/jinlian2.txt
~~~

18.-rmdir：删除空目录

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -mkdir /test
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -rmdir /test
~~~

19.-du统计文件夹的大小信息

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -du -s -h /user/nogc/test
2.7 K  /user/nogc/test

[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -du  -h /user/nogc/test
1.3 K  /user/nogc/test/README.txt
15     /user/nogc/test/jinlian.txt
1.4 K  /user/nogc/test/zaiyiqi.txt
~~~

20.-setrep：设置HDFS中文件的副本数量

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -setrep 5 /README.txt
~~~

这里设置的副本数只是记录在NameNode的元数据中，是否真的会有这么多副本，还得看DataNode的数量。因为目前只有3台设备，最多也就3个副本，只有节点数的增加到至少5台时，副本数才能达到5

## 四、HDFS客户端操作（重点）

### 4.1 HDFS客户端环境准备

**1.做准备好Windows依赖版 Hadoop-3.1.0**

**2.在Windows上配置HADOOP_HOME环境变量**

具体设置根据自己安装目录自行调节

~~~shell
如：变量名：HADOOP_HOME
   变量值：E:\hadoop\hadoop-3.1.0
~~~

**3.配Path环境变量**

~~~powershell
变量名：Path
变量值：%HADOOP_HOME%\bin
~~~



### 4.2 创建Maven工程，并导入相关依赖+日志

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

在项目的src/main /resources目录下，新建一个文件，命名为“log4j2.xml”，在文件中填入

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



### 4.3 创建包与HdfsClient类

~~~java
public class HdfsClient{	
   @Test
    public void testHdfsClient() throws IOException, InterruptedException {
        //1. 创建HDFS客户端对象,传入uri， configuration , user
        FileSystem fileSystem =
                FileSystem.get(URI.create("hdfs://hadoop102:9820"), new Configuration(), "nogc");
        //2. 操作集群
        // 例如：在集群的/目录下创建 testHDFS目录
        fileSystem.mkdirs(new Path("/testHDFS"));
        //3. 关闭资源
        fileSystem.close();
    }
}
~~~



### 4.4 执行程序



## 五、HDFS的API操作

### 5.1 HDFS文件上传（测试参数优先级）

**1.编写源代码**

~~~java
@Test
public void testCopyFromLocalFile() throws IOException, InterruptedException, URISyntaxException {

		// 1 获取文件系统
		Configuration configuration = new Configuration();
        // 设置副本数为2个
		configuration.set("dfs.replication", "2");
		FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"), configuration, "nogc");

		// 2 上传文件
		fs.copyFromLocalFile(new Path("e:/banzhang.txt"), new Path("/banzhang.txt"));

		// 3 关闭资源
		fs.close();

}
~~~

**2.在项目的resources中新建hdfs-site.xml文件，并将如下内容拷贝进去，再次测试**

~~~java
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<property>
		<name>dfs.replication</name>
        <value>1</value>
	</property>
</configuration>
~~~

**3.参数优先级**

~~~java
参数优先级排序：（1）客户端代码中设置的值 >（2）ClassPath下的用户自定义配置文件 >（3）然后是服务器的自定义配置(xxx-site.xml) >（4）服务器的默认配置(xxx-default.xml)
~~~



### 5.2 HDFS文件下载

~~~java
@Test
public void testCopyToLocalFile() throws IOException, InterruptedException, URISyntaxException{

		// 1 获取文件系统
		Configuration configuration = new Configuration();
		FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"), configuration, "nogc");
		
		// 2 执行下载操作
		// boolean delSrc 指是否将原文件删除
		// Path src 指要下载的文件路径
		// Path dst 指将文件下载到的路径
		// boolean useRawLocalFileSystem 是否开启文件校验
		fs.copyToLocalFile(false, new Path("/banzhang.txt"), new Path("e:/banhua.txt"), true);
		
		// 3 关闭资源
		fs.close();
}
~~~



### 5.3 HDFS文件夹删除

~~~java
@Test
public void testDelete() throws IOException, InterruptedException, URISyntaxException{

	// 1 获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"), configuration, "nogc");
		
	// 2 执行删除
	fs.delete(new Path("/0213/"), true);
		
	// 3 关闭资源
	fs.close();
}
~~~



### 5.4 HDFS文件名更改/移动

~~~java
@Test
public void testRename() throws IOException, InterruptedException, URISyntaxException{

	// 1 获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"), configuration, "nogc"); 
		
	// 2 修改文件名称
	fs.rename(new Path("/banzhang.txt"), new Path("/banhua.txt"));
		
	// 3 关闭资源
	fs.close();
}
~~~



### 5.5 HDFS文件详情查看

查看文件名称、权限、长度、块信息

~~~java
@Test
public void testListFiles() throws IOException, InterruptedException, URISyntaxException{

	// 1获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"), configuration, "nogc"); 
		
	// 2 获取文件详情
	RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);
		
	while(listFiles.hasNext()){
		LocatedFileStatus status = listFiles.next();
			
		// 输出详情
		// 文件名称
		System.out.println(status.getPath().getName());
		// 长度
		System.out.println(status.getLen());
		// 权限
		System.out.println(status.getPermission());
		// 分组
		System.out.println(status.getGroup());
			
		// 获取存储的块信息
		BlockLocation[] blockLocations = status.getBlockLocations();
			
		for (BlockLocation blockLocation : blockLocations) {
				
			// 获取块存储的主机节点
			String[] hosts = blockLocation.getHosts();
				
			for (String host : hosts) {
				System.out.println(host);
			}
		}
			
		System.out.println("-----------漂亮的分割线----------");
	}

// 3 关闭资源
fs.close();
}
~~~



### 5.6 HDFS文件和文件夹判断

~~~java
@Test
public void testListStatus() throws IOException, InterruptedException, URISyntaxException{
		
	// 1 获取文件配置信息
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"), configuration, "nogc");
		
	// 2 判断是文件还是文件夹
	FileStatus[] listStatus = fs.listStatus(new Path("/"));
		
	for (FileStatus fileStatus : listStatus) {
		
		// 如果是文件
		if (fileStatus.isFile()) {
				System.out.println("f:"+fileStatus.getPath().getName());
			}else {
				System.out.println("d:"+fileStatus.getPath().getName());
			}
		}
		
	// 3 关闭资源
	fs.close();
}
~~~



## 六、HDFS的数据流（重点）

### 6.1 HDFS写数据流

##### **1.剖析文件写入**

![HDFS4-1](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS4-1.png)

（1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。

（2）NameNode返回是否可以上传。

（3）客户端请求第一个 Block上传到哪几个DataNode服务器上。

（4）NameNode返回3个DataNode节点，分别为dn1、dn2、dn3。

（5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。

（6）dn1、dn2、dn3逐级应答客户端。

（7）客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。

（8）当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。



##### **2.网络拓扑-节点距离计算**

在HDFS写数据的过程中，NameNode会选择距离待上传数据最近距离的DataNode接收数据。那么这个最近距离怎么计算呢？

**节点距离：两个节点到达最近的共同祖先的距离总和。**

例如，假设有数据中心d1机架r1中的节点n1。该节点可以表示为/d1/r1/n1。利用这种标记，这里给出四种距离描述，如图:

![HDFS4-2](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS4-2.png)



##### 3.机架感知（副本存储节点选择）

说明：

~~~shell
For the common case, when the replication factor is three, HDFS’s placement policy is to put one replica on the local machine if the writer is on a datanode, otherwise on a random datanode, another replica on a node in a different (remote) rack, and the last on a different node in the same remote rack.
~~~

Hadoop3.1.3副本节点选择

![HDFS4-4](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS4-4.png)



### 6.2 HDFS读数据流程

HDFS的读数据流程，如图：

![HDFS4-5](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS4-5.png)

（1）客户端通过Distributed FileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。

（2）挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。

（3）DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。

（4）客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。



## 七、NameNode和SecondaryNameNode（重点）

### 7.1 NN和2NN工作机制

**思考：NameNode中的元数据是存储在哪里的？**

首先，我们做个假设，如果存储在NameNode节点的磁盘中，因为经常需要进行随机访问，还有响应客户请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的FsImage。

这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新FsImage，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦NameNode节点断电，就会产生数据丢失。因此，引入Edits文件(只进行追加操作，效率很高)。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到Edits中。这样，一旦NameNode节点断电，可以通过FsImage和Edits的合并，合成元数据。

但是，如果长时间添加数据到Edits中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行FsImage和Edits的合并，如果这个操作由NameNode节点完成，又会效率过低。因此，引入一个新的节点SecondaryNamenode，专门用于FsImage和Edits的合并。

NN和2NN工作机制，如图:

![HDFS5-1](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS5-1.png)

1. **第一阶段：NameNode启动**

（1）第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。

（2）客户端对元数据进行增删改的请求。

（3）NameNode记录操作日志，更新滚动日志。

（4）NameNode在内存中对元数据进行增删改。

2. **第二阶段：Secondary NameNode工作**

​    （1）Secondary NameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否检查结果。

​    （2）Secondary NameNode请求执行CheckPoint。

​    （3）NameNode滚动正在写的Edits日志。

​    （4）将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。

​    （5）Secondary NameNode加载编辑日志和镜像文件到内存，并合并。

​    （6）生成新的镜像文件fsimage.chkpoint。

​    （7）拷贝fsimage.chkpoint到NameNode。

​    （8）NameNode将fsimage.chkpoint重新命名成fsimage。

~~~shell
NN和2NN工作机制详解：
Fsimage：NameNode内存中元数据序列化后形成的文件。
Edits：记录客户端更新元数据信息的每一步操作（可通过Edits运算出元数据）。
NameNode启动时，先滚动Edits并生成一个空的edits.inprogress，然后加载Edits和Fsimage到内存中，此时NameNode内存就持有最新的元数据信息。Client开始对NameNode发送元数据的增删改的请求，这些请求的操作首先会被记录到edits.inprogress中（查询元数据的操作不会被记录在Edits中，因为查询操作不会更改元数据信息），如果此时NameNode挂掉，重启后会从Edits中读取元数据的信息。然后，NameNode会在内存中执行元数据的增删改的操作。
由于Edits中记录的操作会越来越多，Edits文件会越来越大，导致NameNode在启动加载Edits时会很慢，所以需要对Edits和Fsimage进行合并（所谓合并，就是将Edits和Fsimage加载到内存中，照着Edits中的操作一步步执行，最终形成新的Fsimage）。SecondaryNameNode的作用就是帮助NameNode进行Edits和Fsimage的合并工作。
SecondaryNameNode首先会询问NameNode是否需要CheckPoint（触发CheckPoint需要满足两个条件中的任意一个，定时时间到和Edits中数据写满了）。直接带回NameNode是否检查结果。SecondaryNameNode执行CheckPoint操作，首先会让NameNode滚动Edits并生成一个空的edits.inprogress，滚动Edits的目的是给Edits打个标记，以后所有新的操作都写入edits.inprogress，其他未合并的Edits和Fsimage会拷贝到SecondaryNameNode的本地，然后将拷贝的Edits和Fsimage加载到内存中进行合并，生成fsimage.chkpoint，然后将fsimage.chkpoint拷贝给NameNode，重命名为Fsimage后替换掉原来的Fsimage。NameNode在启动时就只需要加载之前未合并的Edits和Fsimage即可，因为合并过的Edits中的元数据信息已经被记录在Fsimage中。
~~~



### 7.2 Fsimage和Edits解析

**1.概念**

![HDFS5-2](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS5-2.png)

**2.oiv查看Fsimage文件**

(1) 查看oiv和oev命令

~~~shell
[nogc@hadoop102 current]$ hdfs
oiv            apply the offline fsimage viewer to an fsimage
oev            apply the offline edits viewer to an edits file
~~~

(2) 基本语法

hdfs oiv -p 文件类型 -i镜像文件 -o 转换后文件输出路径

(3) 案例实操

~~~shell
[nogc@hadoop102 current]$ pwd
/opt/module/hadoop-3.1.3/data/tmp/dfs/name/current

[nogc@hadoop102 current]$ hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-3.1.3/fsimage.xml

[nogc@hadoop102 current]$ cat /opt/module/hadoop-3.1.3/fsimage.xml
~~~

将显示的xml文件内容拷贝到Eclipse中创建的xml文件中，并格式化。部分显示结果如下:

~~~shell
<inode>
	<id>16386</id>
	<type>DIRECTORY</type>
	<name>user</name>
	<mtime>1512722284477</mtime>
	<permission>nogc:supergroup:rwxr-xr-x</permission>
	<nsquota>-1</nsquota>
	<dsquota>-1</dsquota>
</inode>
<inode>
	<id>16387</id>
	<type>DIRECTORY</type>
	<name>nogc</name>
	<mtime>1512790549080</mtime>
	<permission>nogc:supergroup:rwxr-xr-x</permission>
	<nsquota>-1</nsquota>
	<dsquota>-1</dsquota>
</inode>
<inode>
	<id>16389</id>
	<type>FILE</type>
	<name>wc.input</name>
	<replication>3</replication>
	<mtime>1512722322219</mtime>
	<atime>1512722321610</atime>
	<perferredBlockSize>134217728</perferredBlockSize>
	<permission>nogc:supergroup:rw-r--r--</permission>
	<blocks>
		<block>
			<id>1073741825</id>
			<genstamp>1001</genstamp>
			<numBytes>59</numBytes>
		</block>
	</blocks>
</inode >
~~~

思考：可以看出，Fsimage中没有记录块所对应DataNode，为什么？

在集群启动后，要求DataNode上报数据块信息，并间隔一段时间后再次上报。

**3. oev查看Edits文件**

1.基本语法

~~~shell
hdfs oev -p 文件类型 -i编辑日志 -o 转换后文件输出路径
~~~

2.案例实操

~~~shell
[nogc@hadoop102 current]$ hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-3.1.3/edits.xml

[nogc@hadoop102 current]$ cat /opt/module/hadoop-3.1.3/edits.xml
~~~

将显示的xml文件内容拷贝到Eclipse中创建的xml文件中，并格式化。显示结果如下:

~~~shell
<?xml version="1.0" encoding="UTF-8"?>
<EDITS>
	<EDITS_VERSION>-63</EDITS_VERSION>
	<RECORD>
		<OPCODE>OP_START_LOG_SEGMENT</OPCODE>
		<DATA>
			<TXID>129</TXID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_ADD</OPCODE>
		<DATA>
			<TXID>130</TXID>
			<LENGTH>0</LENGTH>
			<INODEID>16407</INODEID>
			<PATH>/hello7.txt</PATH>
			<REPLICATION>2</REPLICATION>
			<MTIME>1512943607866</MTIME>
			<ATIME>1512943607866</ATIME>
			<BLOCKSIZE>134217728</BLOCKSIZE>
			<CLIENT_NAME>DFSClient_NONMAPREDUCE_-1544295051_1</CLIENT_NAME>
			<CLIENT_MACHINE>192.168.1.5</CLIENT_MACHINE>
			<OVERWRITE>true</OVERWRITE>
			<PERMISSION_STATUS>
				<USERNAME>nogc</USERNAME>
				<GROUPNAME>supergroup</GROUPNAME>
				<MODE>420</MODE>
			</PERMISSION_STATUS>
			<RPC_CLIENTID>908eafd4-9aec-4288-96f1-e8011d181561</RPC_CLIENTID>
			<RPC_CALLID>0</RPC_CALLID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_ALLOCATE_BLOCK_ID</OPCODE>
		<DATA>
			<TXID>131</TXID>
			<BLOCK_ID>1073741839</BLOCK_ID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_SET_GENSTAMP_V2</OPCODE>
		<DATA>
			<TXID>132</TXID>
			<GENSTAMPV2>1016</GENSTAMPV2>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_ADD_BLOCK</OPCODE>
		<DATA>
			<TXID>133</TXID>
			<PATH>/hello7.txt</PATH>
			<BLOCK>
				<BLOCK_ID>1073741839</BLOCK_ID>
				<NUM_BYTES>0</NUM_BYTES>
				<GENSTAMP>1016</GENSTAMP>
			</BLOCK>
			<RPC_CLIENTID></RPC_CLIENTID>
			<RPC_CALLID>-2</RPC_CALLID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_CLOSE</OPCODE>
		<DATA>
			<TXID>134</TXID>
			<LENGTH>0</LENGTH>
			<INODEID>0</INODEID>
			<PATH>/hello7.txt</PATH>
			<REPLICATION>2</REPLICATION>
			<MTIME>1512943608761</MTIME>
			<ATIME>1512943607866</ATIME>
			<BLOCKSIZE>134217728</BLOCKSIZE>
			<CLIENT_NAME></CLIENT_NAME>
			<CLIENT_MACHINE></CLIENT_MACHINE>
			<OVERWRITE>false</OVERWRITE>
			<BLOCK>
				<BLOCK_ID>1073741839</BLOCK_ID>
				<NUM_BYTES>25</NUM_BYTES>
				<GENSTAMP>1016</GENSTAMP>
			</BLOCK>
			<PERMISSION_STATUS>
				<USERNAME>nogc</USERNAME>
				<GROUPNAME>supergroup</GROUPNAME>
				<MODE>420</MODE>
			</PERMISSION_STATUS>
		</DATA>
	</RECORD>
</EDITS >
~~~



### 7.3 CheckPoint时间设置

**1.通常情况下，SecondaryNameNode每隔一小时执行一次。**

[hdfs-default.xml]

~~~shell
<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>3600</value>
</property>
~~~

**2.一分钟检查一次操作次数，3当操作次数达到1百万时，SecondaryNameNode执行一次。**

~~~shell
<property>
  <name>dfs.namenode.checkpoint.txns</name>
  <value>1000000</value>
<description>操作动作次数</description>
</property>

<property>
  <name>dfs.namenode.checkpoint.check.period</name>
  <value>60</value>
<description> 1分钟检查一次操作次数</description>
</property >
~~~



### 7.4 集群安全模式

集群处于安全模式，不能执行重要操作（写操作）。集群启动完成后，自动退出安全模式。

~~~shell
bin/hdfs dfsadmin -safemode get		（功能描述：查看安全模式状态）

bin/hdfs dfsadmin -safemode enter  	（功能描述：进入安全模式状态）

bin/hdfs dfsadmin -safemode leave	（功能描述：离开安全模式状态）

bin/hdfs dfsadmin -safemode wait	（功能描述：等待安全模式状态）
~~~

**模拟等待安全模式案例**

1.查看当前模式

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hdfs dfsadmin -safemode get
Safe mode is OFF
~~~

2.先进入安全模式

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ bin/hdfs dfsadmin -safemode enter
~~~

3.创建并执行下面的脚本

在/opt/module/hadoop-3.1.3路径上，编辑一个脚本safemode.sh

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ touch safemode.sh
[nogc@hadoop102 hadoop-3.1.3]$ vim safemode.sh

#!/bin/bash
hdfs dfsadmin -safemode wait
hdfs dfs -put /opt/module/hadoop-3.1.3/README.txt /

[nogc@hadoop102 hadoop-3.1.3]$ chmod 777 safemode.sh

[nogc@hadoop102 hadoop-3.1.3]$ ./safemode.sh
~~~

4.再打开一个窗口，执行

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ bin/hdfs dfsadmin -safemode leave
~~~

5.观察

（a）再观察上一个窗口

​		  Safe mode is OFF

（b）HDFS集群上已经有上传的数据了



### 7.4 NameNode多目录配置

1.NameNode的本地目录可以配置成多个，且每个目录存放内容相同，增加了可靠性

2.具体配置如下

（1）在hdfs-site.xml文件中修改如下内容

~~~shell
<property>
    <name>dfs.namenode.name.dir</name>
<value>file:///${hadoop.tmp.dir}/name1,file:///${hadoop.tmp.dir}/name2</value>
</property>
~~~

（2）停止集群，删除data和logs中所有数据

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ rm -rf data/ logs/
[nogc@hadoop103 hadoop-3.1.3]$ rm -rf data/ logs/
[nogc@hadoop104 hadoop-3.1.3]$ rm -rf data/ logs/
~~~

（3）格式化集群并启动

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ bin/hdfs namenode –format
[nogc@hadoop102 hadoop-3.1.3]$ sbin/start-dfs.sh
~~~

（4）查看结果

~~~shell
[nogc@hadoop102 dfs]$ ll
总用量 12
drwx------. 3 nogc nogc 4096 12月 11 08:03 data
drwxrwxr-x. 3 nogc nogc 4096 12月 11 08:03 name1
drwxrwxr-x. 3 nogc nogc 4096 12月 11 08:03 name2
~~~



## 八、DataNode(重点)

### 8.1 DataNode工作机制

![HDFS6-1](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS6-1.png)

(1）一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

(2）DataNode启动后向NameNode注册，通过后，周期性（1小时）的向NameNode上报所有的块信息。

(3）心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。

(4）集群运行中可以安全加入和退出一些机器。



### 8.2 数据完整性

思考：如果电脑磁盘里面存储的数据是控制高铁信号灯的红灯信号（1）和绿灯信号（0），但是存储该数据的磁盘坏了，一直显示是绿灯，是否很危险？同理DataNode节点上的数据损坏了，却没有发现，是否也很危险，那么如何解决呢？

如下是DataNode节点保证数据完整性的方法。

1）当DataNode读取Block的时候，它会计算CheckSum。

2）如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。

3）Client读取其他DataNode上的Block。

4）常见的校验算法 crc（32）  md5（128）  sha1（160）

5）DataNode在其文件创建后周期验证CheckSum，如图

![HDFS6-2](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS6-2.png)



### 8.3 掉线时限参数设置

![HDFS6-3](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS6-3.png)

需要注意的是hdfs-site.xml 配置文件中的heartbeat.recheck.interval的单位为毫秒，dfs.heartbeat.interval的单位为秒。

~~~shell
<property>
    <name>dfs.namenode.heartbeat.recheck-interval</name>
    <value>300000</value>
</property>
<property>
    <name>dfs.heartbeat.interval</name>
    <value>3</value>
</property>
~~~



### 8.4 服役新数据节点

随着公司业务的增长，数据量越来越大，原有的数据节点的容量已经不能满足存储数据的需求，需要在原有集群基础上动态添加新的数据节点。

**1.环境准备**

（1）在hadoop104主机上再克隆一台hadoop105主机

（2）修改IP地址和主机名称

（3）**删除原来HDFS文件系统留存的文件（/opt/module/hadoop-3.1.3/data和log）**

（4）source一下配置文件

~~~shell
[nogc@hadoop105 hadoop-3.1.3]$ source /etc/profile
~~~

**2.服役新节点具体步骤**

（1）直接启动DataNode，即可关联到集群

~~~shell
[nogc@hadoop105 hadoop-3.1.3]$ hdfs --daemon start datanode
[nogc@hadoop105 hadoop-3.1.3]$yarn -–daemon start nodemanager
~~~

![HDFS6-4](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS6-4.png)



（2）如果数据不均衡，可以用命令实现集群的再平衡

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ sbin/start-balancer.sh
~~~



### 8.5 退役旧数据节点

##### **1.添加白名单 和 黑名单**

添加到白名单的主机节点，都允许访问NameNode，不在白名单的主机节点，都会被直接退出。

​    添加到黑名单的主机节点，不允许访问NameNode，会在数据迁移后退出。

​    实际情况下，白名单用于确定允许访问NameNode的DataNode节点，内容配置一般与workers文件内容一致。 黑名单用于在集群运行过程中退役DataNode节点。

(1）在NameNode的/opt/module/hadoop-3.1.3/etc/hadoop目录下分别创建whitelist 和blacklist文件

~~~shell
[nogc@hadoop102 hadoop]$ pwd
/opt/module/hadoop-3.1.3/etc/hadoop
[nogc@hadoop102 hadoop]$ touch whitelist
[nogc@hadoop102 hadoop]$ touch blacklist
~~~

在whitelist中添加如下主机名称,假如集群正常工作的节点为102 103 104 105 

~~~shell
hadoop102
hadoop103
hadoop104
hadoop105
~~~

黑名单暂时为空。

（2）在NameNode的hdfs-site.xml配置文件中增加dfs.hosts 和 dfs.hosts.exclude配置

~~~shell
<property>
<name>dfs.hosts</name>
<value>/opt/module/hadoop-3.1.3/etc/hadoop/whitelist</value>
</property>

<property>
<name>dfs.hosts.exclude</name>
<value>/opt/module/hadoop-3.1.3/etc/hadoop/blacklist</value>
</property>
~~~

（3）配置文件分发

~~~shell
[nogc@hadoop102 hadoop]$ xsync hdfs-site.xml
~~~

（4）重新启动集群

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ stop-dfs.sh
[nogc@hadoop102 hadoop-3.1.3]$ start-dfs.sh
注意: 因为workers中没有配置105,需要单独在105启动DN
~~~

（5）在web端查看目前正常工作的DN节点

![HDFS6-5](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS6-5.png)

##### **2.黑名单退役**

（1）准备使用黑名单退役105,编辑blacklist文件，添加105

~~~shell
[nogc@hadoop102 hadoop] vim blacklist
hadoop105
~~~

（2）刷新NameNode

~~~shell
[nogc@hadoop102 hadoop] hdfs dfsadmin -refreshNodes
~~~

（3）在web端查看DN状态，105 正在退役中…进行数据的迁移

![HDFS6-6](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS6-6.png)

（4）如果105也启动的NodeManager，也可以刷新yarn状态。【可选查看】

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ yarn rmadmin -refreshNodes
~~~



##### 3.白名单退役[不推荐]

白名单退役会直接将节点抛弃，没有迁移数据的过程，会造成数据丢失。

（1）   删除blacklist的中的内容，恢复 102 103 104 105 正常工作，如图退役前DataNode节点

![HDFS6-7](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS6-7.png)

（2）修改whitelist，将105删除,保留102 103 104

~~~shell
[nogc@hadoop102 hadoop]$ vim whitelist
hadoop102
hadoop103
hadoop104
~~~

（3）刷新NameNode

~~~shell
[nogc@hadoop102 hadoop]$ vim whitelist
hadoop102
hadoop103
hadoop104
~~~

（4）web端查看，发现105节点直接从集群列表中丢弃

![HDFS6-8](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS6-8.png)



### 8.6 Datanode多目录配置

**1.DataNode也可以配置成多个目录，每个目录存储的数据不一样。即：数据不是副本**

**2.具体配置如下**

（1）在hdfs-site.xml中修改如下内容:

~~~shell
<property>
        <name>dfs.datanode.data.dir</name>
<value>file:///${hadoop.tmp.dir}/data1,file:///${hadoop.tmp.dir}/data2</value>
</property>
~~~

（2）停止集群，删除data和logs中所有数据。

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ rm -rf data/ logs/
[nogc@hadoop103 hadoop-3.1.3]$ rm -rf data/ logs/
[nogc@hadoop104 hadoop-3.1.3]$ rm -rf data/ logs/
~~~

（3）格式化集群并启动。

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ bin/hdfs namenode –format
[nogc@hadoop102 hadoop-3.1.3]$ sbin/start-dfs.sh
~~~

（4）查看结果

~~~shell
[nogc@hadoop102 dfs]$ ll
总用量 12
drwx------. 3 nogc nogc 4096 4月   4 14:22 data1
drwx------. 3 nogc nogc 4096 4月   4 14:22 data2
drwxrwxr-x. 3 nogc nogc 4096 12月 11 08:03 name1
drwxrwxr-x. 3 nogc nogc 4096 12月 11 08:03 name2
~~~



## 九、小文件存档

![HDFS6-9](https://github.com/bigdata2018/BigData-Hadoop/blob/master/picture/HDFS6-9.png)

**3．案例实操**

（1）需要启动YARN进程

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ start-yarn.sh
~~~

（2）归档文件

把/user/nogc/input目录里面的所有文件归档成一个叫input.har的归档文件，并把归档后文件存储到/user/nogc/output路径下。

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ bin/hadoop archive -archiveName input.har –p  /user/nogc/input   /user/nogc/output
~~~

（3）查看归档

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -lsr /user/nogc/output/input.har
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -lsr har:///user/nogc/output/input.har
~~~

（4）解归档文件

~~~shell
[nogc@hadoop102 hadoop-3.1.3]$ hadoop fs -cp har:/// user/nogc/output/input.har/*    /user/nogc
~~~





## 参考资料

1. [Apache Hadoop 3.1.3 > HDFS Architecture](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
2. 尚硅谷大数据文档
