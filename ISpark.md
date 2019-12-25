## 安装部署
```
	cat >>/home/docker/software/spark-1.5.0-bin-hadoop2.6/conf/spark-env.sh<<EOF
	HADOOP_CONF_DIR=/home/docker/software/hadoop-2.6.0/etc/hadoop
	EOF
```
### 启动pi
```
./spark-submit --master yarn --class  org.apache.spark.examples.SparkPi /home/bjrdc/software/spark-2.3.1-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.3.1.jar
```
### 启动Spark wordcount
```
	/home/docker/software/spark-1.5.0/bin/spark-submit --master yarn-cluster --class  com.xjgz.cdc.spark.mr.HdfsWordCount  /home/docker/mrs/spark-mapreduce-0.0.1-SNAPSHOT-package.jar
	./spark-submit --master yarn-cluster --class  com.xjgz.cdc.spark.mr.HdfsWordCount  /home/solar/software/spark-2.3.1-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.3.1.jar 
```
### Pi
```
	./spark-submit --master yarn-cluster --class org.apache.spark.examples.SparkPi   /home/solar/software/spark-2.3.1-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.3.1.jar 10     
```
### jmx
在执行spark-commit 的时候带上如下参数，用于监控executor
```
--conf "spark.executor.extraJavaOptions=-XX:+UnlockCommercialFeatures -XX:+FlightRecorder -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.port=0"
```
如果端口为0，则自动选择端口
```
--conf "spark.driver.extraJavaOptions=-XX:+UnlockCommercialFeatures -XX:+FlightRecorder -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.port=0"
```
### spark2.1.0 with hadoop-2.6.0
```
cat >>/home/docker/software/spark-1.5.0-bin-hadoop2.6/conf/spark-env.sh<<EOF
	HADOOP_CONF_DIR=/home/docker/software/hadoop-2.6.0/etc/hadoop
	EOF
```

### 问题处理
A、FetchFailedException:
	
B、sizeexceeds interget.MAC_VALUE
 因为Spark shffle的时候用的bufferwrite，这个对象最大是整形的。通过rdd.repartition(5000)这样的方式规避
C、Spark Scala throws “java.util.NoSuchElementException:
 https://issues.apache.org/jira/browse/SPARK-16473
 升级到2.1.1试试

### driver上执行的方法
A、parallelizing an existing collection in your driver program
B、actions, which return a value to the driver program after running a computation on the dataset. 

### 编译
```
export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=512m"
	mvn -Pyarn -Phadoop-2.3 -Dhadoop.version=2.3.0 -DskipTests clean package
```
### log4j
```
	vi ${spark_home}/conf/log4j.properties
```
### slf4j
SLF4J: Found binding in [jar:file:/cloud/HDFS-YARN-TMP/nm-local-dir/usercache/bjrdc/filecache/16/__spark_libs__6014693449407005386.zip/slf4j-log4j12-1.7.16.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/bjrdc/software/hadoop-3.0.3/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]


### 异常处理
	Failed to get broadcast_1_piece0 of broadcast_1，可能是在闭包或者地柜中使用了rdd，修改为循环即可

# Spark-Streaming
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
```
		cat >>/home/docker/spark-1.5.0/conf/spark-env.sh<<EOF
			HADOOP_CONF_DIR=/home/docker/software/hadoop-2.6.0/etc/hadoop
		EOF
```
### 执行
	./software/spark-1.5.0/bin/spark-submit --master yarn-cluster --class com.xjgz.cdc.spark.streaming.alarm.PersonAlarm  /home/docker/mrs/spark-streaming-0.0.1-SNAPSHOT-package.jar
	该命令将在hadoop集群上执行com.xjgz.cdc.spark.streaming.alarm.PersonAlarm这个流计算程序
### 异常处理
A、classnotfound org.apache.hadoop.hbase.ipc.RpcControllerFactory
 这个问题是因为Hbase中已经有了Phoniex的jar包，而hadoop启动的时候也加载了所有的Hbase的jar包，即加载了Phoneix的jar。而phoniex的jar中已经有了一个包叫做org.apache.hadoop.hbase.ipc，所以spark执行的时候不会去重新加载hbase-client.jar中的org.apache.hadoop.hbase.ipc
		解决办法
```
		cat>>/home/docker/software/spark-1.5.0/conf/spark-defaults.conf<<EOF

		spark.executor.extraClassPath	   /home/docker/mrs/lib/hbase-client-1.1.2.jar
		EOF
```
B、org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.LeaseExpiredException): No lease on /spark/checkpoint/temp (inode 23828): File does not exist.
 原因是多个application公用了一个checkpoint目录，将每个应用的目录修改成自己的即可。
### 代码
	sparkstream 中最后一定要在foreachRDD中将每个rdd再foreach一下，否则可能不会执行相关的算子
