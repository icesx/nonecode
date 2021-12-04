### 基本概念
	kafka：消息中间件，具有比较好的特性和spark-streaming集成，详细如下：
		A、消费者【custmer】可以编组【groupid】，相同的groupid下的custmer只能有一个接收到topic的消息
		B、groupid在客户端端订阅和发送消息的时候设置
		C、一个消息会发送到所有的订阅了当前topic的groupid下的custmer——类似广播
		D、一个custmer可以订阅多个topic
		E、有offset，可以读取已经发送的消息——这个还没有实验
	spark-stream：基于spark的流计算的框架
### 安装部署
	A、spark安装
		cat >>/home/docker/spark-1.5.0/conf/spark-env.sh<<EOF
			HADOOP_CONF_DIR=/home/docker/software/hadoop-2.6.0/etc/hadoop
		EOF
### 执行
	./software/spark-1.5.0/bin/spark-submit --master yarn-cluster --class com.xjgz.cdc.spark.streaming.alarm.PersonAlarm  /home/docker/mrs/spark-streaming-0.0.1-SNAPSHOT-package.jar
	该命令将在hadoop集群上执行com.xjgz.cdc.spark.streaming.alarm.PersonAlarm这个流计算程序
### 异常处理
	A、classnotfound org.apache.hadoop.hbase.ipc.RpcControllerFactory
		这个问题是因为Hbase中已经有了Phoniex的jar包，而hadoop启动的时候也加载了所有的Hbase的jar包，即加载了Phoneix的jar。而phoniex的jar中已经有了一个包叫做org.apache.hadoop.hbase.ipc，所以spark执行的时候不会去重新加载hbase-client.jar中的org.apache.hadoop.hbase.ipc
		解决办法
		cat>>/home/docker/software/spark-1.5.0/conf/spark-defaults.conf<<EOF
		spark.executor.extraClassPath	   /home/docker/mrs/lib/hbase-client-1.1.2.jar
		EOF
	B、org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.LeaseExpiredException): No lease on /spark/checkpoint/temp (inode 23828): File does not exist.
		原因是多个application公用了一个checkpoint目录，将每个应用的目录修改成自己的即可。
### 代码
	sparkstream 中最后一定要在foreachRDD中将每个rdd再foreach一下，否则可能不会执行相关的算子
