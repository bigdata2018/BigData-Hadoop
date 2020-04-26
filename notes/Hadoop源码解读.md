# Hadoop源码解读

整理中....



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

