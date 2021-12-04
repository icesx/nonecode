###原理
	Yarn的ResourceManger（简称RM）通过逻辑上的队列分配内存，CPU等资源给application，默认情况下RM允许最大AM申请Container资源为8192MB("yarn.scheduler.maximum-allocation-mb")，默认情况下的最小分配资源为1024M("yarn.scheduler.minimum-allocation-mb")，AM只能以增量（"yarn.scheduler.minimum-allocation-mb"）和不会超过("yarn.scheduler.maximum-allocation-mb")的值去向RM申请资源，AM负责将("mapreduce.map.memory.mb")和("mapreduce.reduce.memory.mb")的值规整到能被("yarn.scheduler.minimum-allocation-mb")整除，RM会拒绝申请内存超过8192MB和不能被1024MB整除的资源请求。
	
###配置 hadoop2
	A、hadoop2中不需要修改hadoop-env.sh 和增加mapred.child.java.opts
	B、只需要修改如下几个参数
		 yarn.scheduler.minimum-allocation-mb=2048【container的最小内存】
		 yarn.scheduler.maximum-allocation-mb=6144【container的最大内存】
		 yarn.nodemanager.resource.memory-mb=6144【节点可用内存】
		 mapreduce.map.memory.mb=1024			【map任务的内存】
		 mapreduce.map.java.opts=-Xmx819m		【执行map任务的java命令的内存参数】
		 mapreduce.reduce.memory.mb=2048		【reduce container的内存】
		 mapreduce.reduce.java.opts=-Xmx1638m		【启动reduce的java命令参数】
		 yarn.app.mapreduce.am.resource.mb=1024	【】
		 yarn.app.mapreduce.am.command-opts=-Xmx819m【】
		 mapreduce.task.io.sort.mb=409	

	[mapreduce.map.java.opts<mapreduce.map.memory.mb<mapreduce.reduce.java.opts<yarn.scheduler.minimum-allocation-mb=mapreduce.reduce.memory.mb]
	C、具体算法：
		yarn.scheduler.minimum-allocation-mb×3=yarn.nodemanager.resource.memory-mb=yarn.scheduler.maximum-allocation-mb
		mapreduce.map.java.opts*1.25=mapreduce.map.memory.mb
		mapreduce.map.memory.mb*2=mapreduce.reduce.memory.mb
		mapreduce.reduce.java.opts*1.25=mapreduce.reduce.memory.mb
		yarn.app.mapreduce.am.command-opts*1.25=yarn.app.mapreduce.am.resource.mb
		mapreduce.reduce.memory.mb-mapreduce.reduce.java.opts-1=mapreduce.task.io.sort.mb

###使用脚本生成配置
	#git clone https://github.com/mahadevkonar/ambari-yarn-utils.git
	#python yarn-utils.py -c 2 -m 10 -d 1 -k True


###相关配置
####mapr-site.xml
			<?xml version="1.0"?>
		<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
		<configuration>
		  <property> 
		  <name>mapreduce.framework.name</name> 
		  <value>yarn</value> 
		  </property> 
		<property> 
		      <name>mapreduce.map.memory.mb</name>
		      <value>1024</value>
		 </property>
		<property>
		      <name>mapreduce.reduce.memory.mb</name>
		      <value>1024</value>
		 </property>
		<property> 
		      <name>mapreduce.map.java.opts</name>
		      <value>-Xmx819m</value>
		 </property>
		<property> 
		      <name>mapreduce.reduce.java.opts</name>
		      <value>-Xmx1638m</value>
		 </property>
		<property>
			<name>mapred.child.java.opts</name>
			<value>-Xmx1536m</value>
		</property>
		<property>
			<name>mapreduce.task.io.sort.mb</name>
			<value>409</value>
		</property>


		</configuration>
####yarn-site.xml



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

