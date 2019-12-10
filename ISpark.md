### 安装部署
	cat >>/home/docker/software/spark-1.5.0-bin-hadoop2.6/conf/spark-env.sh<<EOF
	HADOOP_CONF_DIR=/home/docker/software/hadoop-2.6.0/etc/hadoop
	EOF
###启动pi
./spark-submit --master yarn --class  org.apache.spark.examples.SparkPi /home/bjrdc/software/spark-2.3.1-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.3.1.jar
### 启动Spark wordcount
	/home/docker/software/spark-1.5.0/bin/spark-submit --master yarn-cluster --class  com.xjgz.cdc.spark.mr.HdfsWordCount  /home/docker/mrs/spark-mapreduce-0.0.1-SNAPSHOT-package.jar
	./spark-submit --master yarn-cluster --class  com.xjgz.cdc.spark.mr.HdfsWordCount  /home/solar/software/spark-2.3.1-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.3.1.jar 
### Pi
	./spark-submit --master yarn-cluster --class org.apache.spark.examples.SparkPi   /home/solar/software/spark-2.3.1-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.3.1.jar 10     

2018-09-2
### jmx
	在执行spark-commit 的时候带上如下参数，用于监控executor
	--conf "spark.executor.extraJavaOptions=-XX:+UnlockCommercialFeatures -XX:+FlightRecorder -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.port=0"
	如果端口为0，则自动选择端口
	--conf "spark.driver.extraJavaOptions=-XX:+UnlockCommercialFeatures -XX:+FlightRecorder -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.port=0"
				
### spark2.1.0 with hadoop-2.6.0
	A、cat >>/home/docker/software/spark-1.5.0-bin-hadoop2.6/conf/spark-env.sh<<EOF
	HADOOP_CONF_DIR=/home/docker/software/hadoop-2.6.0/etc/hadoop
	EOF
	B、
	
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
	export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=512m"
	mvn -Pyarn -Phadoop-2.3 -Dhadoop.version=2.3.0 -DskipTests clean package

### log4j
	vi ${spark_home}/conf/log4j.properties
### slf4j
SLF4J: Found binding in [jar:file:/cloud/HDFS-YARN-TMP/nm-local-dir/usercache/bjrdc/filecache/16/__spark_libs__6014693449407005386.zip/slf4j-log4j12-1.7.16.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/bjrdc/software/hadoop-3.0.3/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]


### 异常处理
	Failed to get broadcast_1_piece0 of broadcast_1，可能是在闭包或者地柜中使用了rdd，修改为循环即可
	 

##python
###