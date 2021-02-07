hadoop
===

## 安装

### 环境要求

1. ubuntu18.04 

2. java8.0.0+

3. 分区要求

   /dev/sdb1                 98G  218M   93G   1% /cloud
   /dev/sdc1                 49G  1.9G   45G   5% /logs

### 配置

#### hadoop3.0.x

1. 准备环境

   1. 主机与节点之间无密码登录配置好
   2. 服务器分区配置好
   3. java环境安装好

2. 需要修改的文件

   -rw-r--r-- 1 bjrdc bjrdc  2760 Mar 31  2020 mapred-site.xml
   -rw-r--r-- 1 bjrdc bjrdc  3302 Mar 30  2020 yarn-site.xml
   -rw-r--r-- 1 bjrdc bjrdc    56 Dec 11  2019 workers
   -rw-r--r-- 1 bjrdc bjrdc 16567 Dec  2  2019 hadoop-env.sh
   -rw-r--r-- 1 bjrdc bjrdc  1447 Nov 15  2019 core-site.xml

3. 配置文件

   1. core-site.xml

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
      <!--
        Licensed under the Apache License, Version 2.0 (the "License");
        you may not use this file except in compliance with the License.
        You may obtain a copy of the License at
      
          http://www.apache.org/licenses/LICENSE-2.0
      
        Unless required by applicable law or agreed to in writing, software
        distributed under the License is distributed on an "AS IS" BASIS,
        WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
        See the License for the specific language governing permissions and
        limitations under the License. See accompanying LICENSE file.
      -->
      
      <!-- Put site-specific property overrides in this file. -->
      
      <configuration>
      <property> 
         <name>fs.defaultFS</name> 
         <value>hdfs://bjrdc10:9100</value> 
      </property>
      
      <property>
        <name>hadoop.tmp.dir</name>
        <value>/cloud/HDFS-YARN-TMP</value>
      <description>tmm</description> 
      </property>
      <property>
         <name>hadoop.http.staticuser.user</name>
         <value>bjrdc</value>
      </property>
      
       <property>
               //代理的用户是admin
               <name>hadoop.proxyuser.bjrdc.hosts</name>
               <value>*</value>
       </property>
      
       <!-- 允许被Oozie代理的用户组 -->
       <property>
               //允许被Oozie代理的用户组是admin所在的组，admin所在的组，都允许被所有的用户代理
               <name>hadoop.proxyuser.bjrdc.groups</name>
               <value>*</value>
       </property> 
      
      </configuration>
      ```

      

   2. hadoop-env.sh

      ```sh
      export JAVA_HOME=/home/bjrdc/software/jdk1.8.0_131_x64
      export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:`/home/bjrdc/software/hbase-2.0.2/bin/hbase classpath`
      ```

      

   3. workers

      ```txt
      bjrdc12
      bjrdc13
      bjrdc14
      bjrdc15
      bjrdc16
      bjrdc20
      bjrdc21
      ```

      

   4. yarn-site.xml

      ```xml
      <?xml version="1.0"?>
      <!--
        Licensed under the Apache License, Version 2.0 (the "License");
        you may not use this file except in compliance with the License.
        You may obtain a copy of the License at
      
          http://www.apache.org/licenses/LICENSE-2.0
      
        Unless required by applicable law or agreed to in writing, software
        distributed under the License is distributed on an "AS IS" BASIS,
        WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
        See the License for the specific language governing permissions and
        limitations under the License. See accompanying LICENSE file.
      -->
      <configuration>
      <!-- Site specific YARN configuration properties -->
      <property>
          <name>yarn.resourcemanager.address</name>
          <value>bjrdc10:8032</value>
      </property>
      <property>
          <name>yarn.resourcemanager.webapp.address</name>
          <value>bjrdc10:8088</value>
      </property>
      <property>
          <name>yarn.resourcemanager.resource-tracker.address</name>
          <value>bjrdc10:8025</value>
      </property>
      <property>
          <name>yarn.resourcemanager.scheduler.address</name>
          <value>bjrdc10:8030</value>
      </property>
      <!--
      <property>
          <name>yarn.nodemanager.log-dirs</name>
          <value>/logs/yarn/userlogs</value>
      </property>
      -->
      <property>
          <name>mapreduce.job.userlog.retain.hours</name>
          <value>36</value>
      </property>
      
      <property>
          <name>yarn.nodemanager.log-dirs</name>
          <value>/logs/yarn/userlogs</value>
      </property>
      
      <property>
          <name>yarn.log-aggregation-enable</name>
          <value>true</value>
      </property>
      <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
      </property>
      <property>
          <name>yarn.log-aggregation.retain-seconds</name>
          <value>172800</value>
      </property>
      <property>
          <name>yarn.log.server.url</name>
          <value>http://bjrdc10:19888/jobhistory/logs</value>
      </property>
      <property>
          <name>mapreduce.jobhistory.intermediate-done-dir</name>
          <value>/tmp/hadoop-yarn/staging/history/done_intermediate</value>
      </property>
      <property>
          <name>mapreduce.jobhistory.done-dir</name>
          <value>/tmp/hadoop-yarn/staging/history/done</value>
      </property>
      <property>
        <name>yarn.application.classpath</name>
            <value>
                /home/bjrdc/software/hadoop-3.0.3/etc/*,
                /home/bjrdc/software/hadoop-3.0.3/etc/hadoop/*,
                /home/bjrdc/software/hadoop-3.0.3/lib/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/common/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/common/lib/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/mapreduce/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/mapreduce/lib-examples/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/mapreduce/lib/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/hdfs/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/hdfs/lib/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/yarn/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/yarn/lib/*,
                /home/bjrdc/software/hbase-2.0.2/lib/*,
            </value>
      </property>
      <property>
              <name>yarn.scheduler.maximum-allocation-vcores</name>
              <value>6</value>
              <final>false</final>
      </property>
      <property>
              <name>yarn.scheduler.maximum-allocation-mb</name>
              <value>20480</value>
              <final>false</final>
      
      </property>
      
      </configuration>
      ```

      

   5. mapred-site.xml

      ```xml
      <?xml version="1.0"?>
      <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
      <!--
        Licensed under the Apache License, Version 2.0 (the "License");
        you may not use this file except in compliance with the License.
        You may obtain a copy of the License at
      
          http://www.apache.org/licenses/LICENSE-2.0
      
        Unless required by applicable law or agreed to in writing, software
        distributed under the License is distributed on an "AS IS" BASIS,
        WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
        See the License for the specific language governing permissions and
        limitations under the License. See accompanying LICENSE file.
      -->
      
      <!-- Put site-specific property overrides in this file. -->
      
      <configuration>
      <property> 
            <name>mapreduce.jobtracker.address</name>
            <value>bjrdc10</value>
      </property>
      <property> 
            <name>mapreduce.jobhistory.address</name>
            <value>bjrdc10:10020</value>
      </property>
      <property> 
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>bjrdc10:19888</value>
      </property>
      <property>
      <name>mapreduce.jobhistory.intermediate-done-dir</name>
      <value>/tmp/hadoop-yarn/staging/history/done_intermediate</value>
      </property>
      <property>
          <name>mapreduce.jobhistory.done-dir</name>
          <value>/tmp/hadoop-yarn/staging/history/done</value>
      </property>
      <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
      </property>
      <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/home/bjrdc/software/hadoop-3.0.3/</value>
      </property>
      <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/home/bjrdc/software/hadoop-3.0.3/</value>
      </property>
      <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/home/bjrdc/software/hadoop-3.0.3/</value>
      </property>
      <property>
        <name>mapreduce.application.classpath</name>
            <value>
                /home/bjrdc/software/hadoop-3.0.3/etc/*,
                /home/bjrdc/software/hadoop-3.0.3/etc/hadoop/*,
                /home/bjrdc/software/hadoop-3.0.3/lib/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/common/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/common/lib/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/mapreduce/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/mapreduce/lib-examples/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/mapreduce/lib/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/hdfs/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/hdfs/lib/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/yarn/*,
                /home/bjrdc/software/hadoop-3.0.3/share/hadoop/yarn/lib/*,
                /home/bjrdc/software/hbase-2.0.2/lib/*,
            </value>
      </property>
      </configuration>
      ```

      

4. 启动

   ```sh
   ./sbin/start-all.sh
   ```

5. 其他

   如果需要配置spark、phoenix参考官方相关材料

#### 管理界面

#### namenode

```http
http://bjrdc10:50070/dfshealth.html#tab-overview
```



#### application

```http
http://bjrdc10:8088/cluster/cluster
```



#### 自动测试

##### pi

```
./hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar pi 2000 2000
	Job Finished in 407.837 seconds
	Estimated value of Pi is 3.14160400000000000000
hadoop-2.6.0
./hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2000 2000
	Job Finished in 411.218 seconds
	Estimated value of Pi is 3.14160400000000000000
```

## yarn

### 如果需要增加datanode可以在master上执行

	./hadoop-daemons.sh start datanode

### 需要将tmp目录也配置出来如下：

```sh
<property>
  <name>hadoop.tmp.dir</name>
  <value>/TOOLS/HDFS-YARN-TMP</value>
<description>tmm</description> 
</property>
```

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

### 重要命令

	sbin/start-all.sh
	bin/hdfs namenode -format

### 问题处理

#### 权限问题：
	hadoop的客户端程序取的用户名是本机的用户名，如果要修改该用户名，需要
	static{
		System.setProperty("HADOOP_USER_NAME", "docker");
	}
	
	C、yarn启动异常
		java.net.ConnectException: Call From hadoop2/172.16.1.2 to 0.0.0.0:8031 failed on connection exception，这个原因是yarn的默认配置去拿了0.0.0.0为resourcemanager的地址，需要在yarn-site中制定master地址。
	
	E、webUI 无法打卡 connection reset
		yarn.resourcemanager.webapp.address，这个地址不能设置成0.0.0.0,需要设置为master地址

#### yarn mapreduce执行等待时间长【yarn-2.6.0】
​	resucemanage的日志停止在：State change from ACCEPTED to RUNNING
​	请看issues：http://marc.info/?l=hadoop-user&m=138678990501262&w=2
​	修改办法,yarn-site.xml：

```xml
 <property>
     <name>yarn.nodemanager.resource.memory-mb</name>
     <value>2400</value>
</property>
<property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>512</value>
</property>
```

​	但是发现不管用
后来先后修改了openfile，yarn-site.xml声明了resourcemanager等均无效
后经过测试，发现在一台主机上是OK的，于是猜测可能是多主机的时候网络的问题，于是想到了修改路由，但是仍然不行。

最后在测试的时候，mapreduce会跑unknown host的异常，于是想到了查看各个container的host的配置，果然去查看的时候，看到了，在一台机器上产生的hosts文件中未包含另外一台主机的上的container的host【后经查询书籍，发现mapreduce的applicationmaster是先分派到一个node上，然后其再向其他的node启动map和reduce，而如果其他的机器有的host无法解析的时候，就会堵塞一直到网络超时——这个问题困了我2天】

### with spark

	config yarn shuffleservice
	echo "CLASSPATH=${CLASSPATH}:/home/docker/software/spark-2.1.1-bin-hadoop2.6/yarn/spark-2.1.1-yarn-shuffle.jar" >> ~${hadoop-home}/libexec/hadoop-config.sh

### log

+ mapred-site.xml

  ```xml
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
  ```

  

+ yarn-site.xml

  ```xml
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
      <value>http://xxx:19888/jobhistory/logs</value>
  </property>
  ```

  



### 内存配置

#### 原理

​	Yarn的ResourceManger（简称RM）通过逻辑上的队列分配内存，CPU等资源给application，默认情况下RM允许最大AM申请Container资源为8192MB("yarn.scheduler.maximum-allocation-mb")，默认情况下的最小分配资源为1024M("yarn.scheduler.minimum-allocation-mb")，AM只能以增量（"yarn.scheduler.minimum-allocation-mb"）和不会超过("yarn.scheduler.maximum-allocation-mb")的值去向RM申请资源，AM负责将("mapreduce.map.memory.mb")和("mapreduce.reduce.memory.mb")的值规整到能被("yarn.scheduler.minimum-allocation-mb")整除，RM会拒绝申请内存超过8192MB和不能被1024MB整除的资源请求。
​	

#### 配置 hadoop2

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

#### 使用脚本生成配置

```
	#git clone https://github.com/mahadevkonar/ambari-yarn-utils.git
​	#python yarn-utils.py -c 2 -m 10 -d 1 -k True
```

#### 相关配置

1. mapr-site.xml

```xml
<?xml version="1.0"?>
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
			<name>mapreduce.task.io.sort.mb</name>
			<value>409</value>
		</property>
	</configuration>
```

2.  yarn-site.xml


```xml
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

## hadoop3 内存配置

### 相关配置项

1. hadoop3 内置有mapreduce的内存自动推导逻辑，目前测试的情况是没有配置mapreduce的相关内存使用就可以使用

2. 只配置scheduler 相关参数即可

   ```xml
   <property>
           <name>yarn.scheduler.maximum-allocation-vcores</name>
           <value>6</value>
           <final>false</final>
   </property>
   <property>
           <name>yarn.scheduler.maximum-allocation-mb</name>
           <value>20480</value>
           <final>false</final>
   </property>
   ```
   


## HDFS


### 注意事项	

1. 删除文件

   ```
   ./hadoop fs -rm -r -f /tmp/logs/docker/logs/
   ```

   

2. 文件系统检测

   ```sh
     bin/hadoop fsck / -delete
   ```
   
    一般不要去执行-delete，因为hadoop自己会进行分片的复制
   
3. hadoop启动会自动去进行分片的平衡，当删除一个datenode后，或者增加一个datenode后，hadoop会自动将丢失的分片找回来

4. slow readProcessor read fields took 。。。ms

5. There appears to be a gap in the edit log
   -recover

6. 离开safemode

   ```
   hadoop dfsadmin -safemode leave 
   ```

### 设置分片

	$hadoop dfs -setrep -w 2 -R /hbase/data/default/CDC_PARTY_SCORE

### 恢复复文件

```
./hdfs debug recoverLease -path /hbase/oldWALs/pv2-00000000000000000478.log
```



### Slow BlockReceiver write packet to mirror

可能是网线降速了，检查网线是不是变黄了

## Ambari

### 安装