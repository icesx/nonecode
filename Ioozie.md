### 编译问题
	编译的时候，使用如下命令即可
	#./bin/mkdistro.sh -Phadoop-2  -Dtomcat.version=7.0.64 -Dhive.version=2.0.0 -Dhbase.version=1.1.2
	该命令编译的版本会在
	#.//distro/target下，但是有一个问题是，编译好的版本里./lib目录下有一个hadoop-common-2.5.1.jar的问题，如果使用的最终的hadoop的版本是2.6.0，则需要删除这个jar，替换成hadoop-common-2.6.0.jar，否则启动的时候会包找不到方法的错误。

### 安装
	./bin/oozie-setup.sh sharelib create -fs hdfs://hadoop-cx11:9100/oozie
### 启动问题
	A、log4j问题
		MethodNotFoundException org.apache.log4j.config.PropertySetter.activer()V
		删除包log4j-1.2-api-2.4.2.jar，原因是这个包里也有个类org.apache.log4j.config.PropertySetter，但是没有active方法
	B、在启动oozie的时候，由于oozie.war中没有包含log4j的配置文件，故需要将${oozie_home}/conf/oozie-log4j.properties 文件cp 到${oozie_home}/oozie-server/webapps/oozie/WEB-INF/class/	
### extjs
	extjs-2.2的版本，下载完成后，需要将其cp到${oozie_home}/oozie_server/webapps/oozie/，并解压缩
	【按照官网说的放到${oozie_home}/libext不行的】		
### mysql配置
		

### hadoop配置
	${oozie_home}/conf/hadoop-conf/core-site.xml
	<property>
		<name>fs.defaultFS</name>
		<value>hadoop-cx11:9100</value>	
	</property>
### 权限
	user docker is not allowed to impersonate docker
	编辑hadoop/etc/hadoop/core-site.xml 增加如下配置	  
	<property>
	    <name>hadoop.proxyuser.docker.hosts</name>
    		<value>*</value>
	  </property>
	  <property>
	    <name>hadoop.proxyuser.docker.groups</name>
	    <value>docker</value>
	  </property>
### 启动historyserver
	#{hadoop}/sbin/mr-jobhistory-daemon.sh start historyserver
	否则多个action不能先后执行，只执行第一个
### 任务启动失败
	Exception in thread "main" java.io.IOException: Cannot initialize Cluster. Please check your configuration for mapreduce.framework.name and the correspond server addresses.  
	是缺少jar包，但是一定奥把jar包放到${oozie_home}/oozie_server/webapps/oozie/WEB-INF/lib

### 示例
	#./bin/oozie-setup.sh sharelib create -fs hdfs://hadoop-cx11:9100/oozie
	[将lib提交到hdfs]
	#~/software/hadoop-2.6.0/bin/hadoop fs -put examples examples
	将会吧example提交的hadoop上，在hadoop的/user/docker/examples 下
	#./bin/oozie job -oozie http://haoop-cx110:11000/oozie -config examples/apps/mapreduce/job.properties -run
### 调用 任务的时候
	found class jline.terminal but interface was expected。因为hadoop下有一个jline包，hive下也有jline包，版本不一样，而之前在hive中配置了HADOOP_USER_CLASSPATH_FIRST=true，而oozie可能不知道
	没有更好的解决版本，只能删除掉所有节点上的~{hadoop}/share/hadoop/yarn/lib/jline-0.9.94.jar,然后重启yarn

### job dag无法出来的问题
	因为编译的时候用的java8，运行用的java7	
	在${oozie}/conf/oozie-env.sh中添加
	export JAVA_HOME=......


