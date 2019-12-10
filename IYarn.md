### 如果需要增加datanode可以在master上执行
	./hadoop-daemons.sh start datanode
### 需要讲tmp目录也配置出来如下：
	<property>
	  <name>hadoop.tmp.dir</name>
	  <value>/TOOLS/HDFS-YARN-TMP</value>
	<description>tmm</description> 
	</property>
### 配置细节
	A、core.xml
		<configuration>
		   <property>
		   <name>fs.defaultFS</name>
		   <value>hdfs://172.16.1.1:9100</value>
		   </property>
		<property>
		  <name>hadoop.tmp.dir</name>
		  <value>/home/docker/volume/HDFS-YARN-TMP</value>
		<description>tmm</description>
		</property>
		</configuration>
	B、mapred-site.xml
		<configuration>
		  <property> 
		  <name>mapreduce.framework.name</name> 
		  <value>yarn</value> 
		  </property> 
		<property> 
		      <name>mapred.job.tracker</name>
		      <value>172.16.1.1:50010</value>
		 </property>
		</configuration>
	C、hdfs-site.xml
		<configuration>
		 <property> 
		  <name>dfs.namenode.name.dir</name> 
		  <value>/home/docker/volume/HDFS-YARN/name</value>
		  <description>  </description> 
		 </property> 
	
		 <property> 
		  <name>dfs.datanode.data.dir</name> 
		  <value>/home/docker/volume/HDFS-YARN/data</value>
		  <description> </description> 
		 </property> 
	
		 <property> 
		   <name>dfs.replication</name>
		   <value>3</value>
		 </property>
		</configuration>
	D、hadoop-env.sh
		export JAVA_HOME=/TOOLS/jdk1.7.0_71_linux_x64
		export HADOOP_PID_DIR=/bigdata/hadoop/pids
	E、yarn-site.xml
		<property>
			<name>yarn.nodemanager.aux-services</name>
			<value>mapreduce_shuffle,spark_shuffle</value>
		</property>
		<property>
			<name>yarn.nodemanager.aux-services.spark_shuffle.class</name>
		<value>org.apache.spark.network.yarn.YarnShuffleService</value>
		</property>
				 <property>
		  <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
		  <value>org.apache.hadoop.mapred.ShuffleHandler</value>
		 </property>
	F、slavers文件中配置节点名称
	J、需要在节点机器上的/etc/hosts中增加节点的ip和域名
		因为hdfs启动的时候，会登录到节点上启动相关的程序，启动的时候会用ip反解域名，如果没有域名则启动异常
	H、http://10.0.88.40:8088/cluster/cluster，查看集群计算情况
	I、http://10.0.88.40:50070/，hdf信息
### Bug
	yarn2.6上的一个bug，当datanode的hostname相同的时候，http://i:50070/dfshealth.html#tab-datanode 这个页面上显示的datanode就智能显示一个
### 问题处理
	如果出现启动hadoop的classnotfound的问题的话，需要去检查一下是否系统中已经设置了hadoop的环境变量，建议不要设置这些环境变量，统统通过hadoop/etc/hadoop/hadoop-env.sh这个文件来设置
### 重要命令：
	sbin/start-all.sh
	bin/hdfs namenode -format
### 问题处理：
	A、权限问题：
		hadoop的客户端程序取的用户名是本机的用户名，如果要修改该用户名，需要
		static{
			System.setProperty("HADOOP_USER_NAME", "docker");
		}
	
	C、yarn启动异常
		java.net.ConnectException: Call From hadoop2/172.16.1.2 to 0.0.0.0:8031 failed on connection exception，这个原因是yarn的默认配置去拿了0.0.0.0为resourcemanager的地址，需要在yarn-site中制定master地址。
	D、yarn mapreduce执行等待时间长【yarn-2.6.0】
		resucemanage的日志停止在：State change from ACCEPTED to RUNNING
		请看issues：http://marc.info/?l=hadoop-user&m=138678990501262&w=2
		修改办法,yarb-site.xml：
			  <property>
			    <name>yarn.nodemanager.resource.memory-mb</name>
			    <value>2400</value>
			  </property>
			  <property>
			    <name>yarn.scheduler.minimum-allocation-mb</name>
			    <value>512</value>
			  </property>
		但是发现不管用
			后来先后修改了openfile，yarn-site.xml声明了resourcemanager等均无效
		后经过测试，发现在一台主机上是OK的，于是猜测可能是多主机的时候网络的问题，于是想到了修改路由，但是仍然不行
		最后在测试的时候，mapreduce会跑unknown host的异常，于是想到了查看各个container的host的配置，果然去查看的时候，看到了，在一台机器上产生的hosts文件中未包含另外一台主机的上的container的host【后经查询书籍，发现mapreduce的applicationmaster是先分派到一个node上，然后其再向其他的node启动map和reduce，而如果其他的机器有的host无法解析的时候，就会堵塞一直到网络超时——这个问题困了我2天】
	E、webUI 无法打卡 connection reset
		yarn.resourcemanager.webapp.address，这个地址不能设置成0.0.0.0,需要设置为master地址
### start-yarn
	#
### with spark
	config yarn shuffleservice
	echo "CLASSPATH=${CLASSPATH}:/home/docker/software/spark-2.1.1-bin-hadoop2.6/yarn/spark-2.1.1-yarn-shuffle.jar" >> ~${hadoop-home}/libexec/hadoop-config.sh

### log

+ mapred-site.xml
<property> 
      <name>mapreduce.jobtracker.address</name>
      <value>hadoop-cx11</value>
</property>
<property> 
      <name>mapreduce.jobhistory.address</name>
      <value>hadoop-cx11:10020</value>
</property>
<property> 
      <name>mapreduce.jobhistory.webapp.address</name>
      <value>hadoop-cx11:19888</value>
</property>
<property>
    <name>mapreduce.job.userlog.retain.hours</name>
    <value>36</value>
</property>

+ yarn-site.xml
  

<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>
<property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>172800</value>
</property>
<property>
    <name>yarn.log.server.url</name>
    <value>http://solar10:19888/jobhistory/logs</value>
</property>



## 内存配置

### 原理
​	Yarn的ResourceManger（简称RM）通过逻辑上的队列分配内存，CPU等资源给application，默认情况下RM允许最大AM申请Container资源为8192MB("yarn.scheduler.maximum-allocation-mb")，默认情况下的最小分配资源为1024M("yarn.scheduler.minimum-allocation-mb")，AM只能以增量（"yarn.scheduler.minimum-allocation-mb"）和不会超过("yarn.scheduler.maximum-allocation-mb")的值去向RM申请资源，AM负责将("mapreduce.map.memory.mb")和("mapreduce.reduce.memory.mb")的值规整到能被("yarn.scheduler.minimum-allocation-mb")整除，RM会拒绝申请内存超过8192MB和不能被1024MB整除的资源请求。
​	
### 配置 hadoop2
​	A、hadoop2中不需要修改hadoop-env.sh 和增加mapred.child.java.opts
​	B、只需要修改如下几个参数
​		 yarn.scheduler.minimum-allocation-mb=2048【container的最小内存】
​		 yarn.scheduler.maximum-allocation-mb=6144【container的最大内存】
​		 yarn.nodemanager.resource.memory-mb=6144【节点可用内存】
​		 mapreduce.map.memory.mb=1024			【map任务的内存】
​		 mapreduce.map.java.opts=-Xmx819m		【执行map任务的java命令的内存参数】
​		 mapreduce.reduce.memory.mb=2048		【reduce container的内存】
​		 mapreduce.reduce.java.opts=-Xmx1638m		【启动reduce的java命令参数】
​		 yarn.app.mapreduce.am.resource.mb=1024	【】
​		 yarn.app.mapreduce.am.command-opts=-Xmx819m【】
​		 mapreduce.task.io.sort.mb=409	

	[mapreduce.map.java.opts<mapreduce.map.memory.mb<mapreduce.reduce.java.opts<yarn.scheduler.minimum-allocation-mb=mapreduce.reduce.memory.mb]
	C、具体算法：
		yarn.scheduler.minimum-allocation-mb×3=yarn.nodemanager.resource.memory-mb=yarn.scheduler.maximum-allocation-mb
		mapreduce.map.java.opts*1.25=mapreduce.map.memory.mb
		mapreduce.map.memory.mb*2=mapreduce.reduce.memory.mb
		mapreduce.reduce.java.opts*1.25=mapreduce.reduce.memory.mb
		yarn.app.mapreduce.am.command-opts*1.25=yarn.app.mapreduce.am.resource.mb
		mapreduce.reduce.memory.mb-mapreduce.reduce.java.opts-1=mapreduce.task.io.sort.mb

### 使用脚本生成配置
```
	#git clone https://github.com/mahadevkonar/ambari-yarn-utils.git
​	#python yarn-utils.py -c 2 -m 10 -d 1 -k True
```

#### 相关配置

1. mapr-site.xml

```

​			<?xml version="1.0"?>
   ​		<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   ​		<configuration>
   ​		  <property> 
   ​		  <name>mapreduce.framework.name</name> 
   ​		  <value>yarn</value> 
   ​		  </property> 
   ​		<property> 
   ​		      <name>mapreduce.map.memory.mb</name>
   ​		      <value>1024</value>
   ​		 </property>
   ​		<property>
   ​		      <name>mapreduce.reduce.memory.mb</name>
   ​		      <value>1024</value>
   ​		 </property>
   ​		<property> 
   ​		      <name>mapreduce.map.java.opts</name>
   ​		      <value>-Xmx819m</value>
   ​		 </property>
   ​		<property> 
   ​		      <name>mapreduce.reduce.java.opts</name>
   ​		      <value>-Xmx1638m</value>
   ​		 </property>
   ​		<property>
   ​			<name>mapred.child.java.opts</name>
   ​			<value>-Xmx1536m</value>
   ​		</property>
   ​		<property>
   ​			<name>mapreduce.task.io.sort.mb</name>
   ​			<value>409</value>
   ​		</property>


	</configuration>
```

2.  yarn-site.xml


```
	<?xml version="1.0"?>
	
	<configuration>
	
	<!-- Site specific YARN configuration properties -->
	<property>  
		<name>yarn.nodemanager.aux-services</name>  
		<value>mapreduce_shuffle</value>  
	</property>  
	<property>  
		<name>yarn.resourcemanager.address</name>  
		<value>hadoop-xj111:8032</value>  
	</property>  
	
	<property>
		<name>yarn.resourcemanager.webapp.address</name>
		<value>0.0.0.0:8088</value>
	</property>
	
	<property>
	    <name>yarn.resourcemanager.resource-tracker.address</name>
	    <value>hadoop-xj111:8025</value>
	  </property>
	  <property>
	    <name>yarn.resourcemanager.scheduler.address</name>
	    <value>hadoop-xj111:8030</value>
	  </property>
	<property>
	    <name>yarn.nodemanager.resource.memory-mb</name>
	    <value>6144</value>
	  </property>
	
	  <property>
	    <name>yarn.scheduler.minimum-allocation-mb</name>
	    <value>2048</value>
	  </property>
	 
	  <property>
	    <name>yarn.scheduler.maximum-allocation-mb</name>
	    <value>6144</value>
	  </property>


	<property>
	   <name>yarn.nodemanager.vmem-check-enabled</name>
	    <value>false</value>
	    <description>Whether virtual memory limits will be enforced for containers</description>
	  </property>
	 <property>
	   <name>yarn.nodemanager.vmem-pmem-ratio</name>
	    <value>2</value>
	    <description>Ratio between virtual memory to physical memory when setting memory limits for containers</description>
	  </property>
	
	 <property>
	   <name>yarn.nodemanager.log-dirs</name>
	    <value>/logs/yarn/userlogs</value> 
	  </property>


	</configuration>
```
