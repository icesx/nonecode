###安装
	A、配置
		#echo HBASE_HOME=/home/docker/software/hbase-1.1.2 >> ${kylin_home}/bin/header.sh
		#echo HADOOP_HOME=/home/docker/software/hadoop-2.6.0 >> ${kylin_home}/bin/header.sh
		#echo HIVE_HOME=/home/docker/software/apache-hive-2.0.0-bin >> ${kylin_home}/bin/header.sh
		#echo PATH=$PATH:$HBASE_HOME/bin:$HADOOP_HOME/bin:$HIVE_HOME/bin
	B、启动
		#${kylin_home}/bin/kylin.sh start
		http://kylin_host:7070/kylin
		帐号：ADMIN	密码：KYLIN
	C、HIVE 数据库名
		#echo kylin.source.hive.database-for-flat-table=CDC >> 	${kylin_home}/conf/kylin.properties
