###zookeeper安装
	A、配置环境变量
			export JAVA_HOME=/home/docker/software/jdk1.7.0_71_linux_x64
	B、配置zoo.cfg
		dataDir=/home/docker/volume/ZOOKEEPER
		#volume 是在各个虚拟机上的挂在的目录
		# the port at which the clients will connect
		clientPort=2181
		server.1=172.16.1.7:2888:3888
		server.2=172.16.1.8:2888:3888
		server.3=172.16.1.107:2888:3888
	C、创建myid文件
		在server.1上执行如下命令		
		echo 1 >/home/docker/volume/ZOOKEEPER/myid
	D、启动zookeeper
		bin/zkServer.sh start
###Hbase安装
	A、配置 JAVA_HOME
		hbase_env.sh中增加
			export JAVA_HOME=/home/docker/software/jdk1.7.0_71_linux_x64
			export HBASE_MANAGES_ZK=false
	B、配置hbase-site.xml
	
###问题处理
	A、2015-09-24 07:29:36,012 INFO  [regionserver/hadoop5/172.16.1.5:16020] regionserver.HRegionServer: reportForDuty to master=localhost,16000,1443079581248 with port=16020, startcode=1443079582628
	修改主机hostname 为实地host，并和/etc/hosts中的ip地址对应
 	B、 regionserver running as process
	C、org.apache.zookeeper.KeeperException$SessionExpiredException: KeeperErrorCode = Session expired
		 <property>
                      <name>zookeeper.session.timeout</name>
                      <value>1200000</value>
                   </property>
                   <property>
                      <name>hbase.zookeeper.property.tickTime</name>
                      <value>6000</value>
                   </property>
	D、如果有一些region没有online的时候
		#./hbase hbck -repair
		如果不行的话，再查看master的日志。如果出现：java.io.IOException: error or interrupted while splitting logs in。这样的异常，那么就可能是异常关集群引起的，修复办法是：
		#./hadoop fs -rm -r /hbase/WALs/hadoop-xj166,16020,1472551606664-splitting
###在进行Hbase的的版本切换的时候，要删除Hadoop Hdfs中的hbase目录中的文件，否则出现版本兼容问题。
###Hbase shell
	create  'xjgz_portal',{NAME=>'portal'}
	put 'PHOENIX_TEST',"\x80\x00\x00\x00",'0:newcolumn','nnnnneeeeewwwww'
	get 'PHOENIX_TEST',"\x80\x00\x00\x00"	【"\x80\x00\x00\x00" 为rowkey】
	scan 'PHOENIX_TEST',{LIMIT=>5}
	create 'KeyPerson','CDC'
	put 'KeyPerson','610123190011234624','CDC:name','zhangzongchang'
###Hadoop中增加Hbase的环境变量
	vi hadoop/etc/hadoop/hadoop-env.sh
	export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:`/home/docker/software/hbase-1.1.2/bin/hbase classpath`	
###单独启动region
	./hbase-daemon.sh start regionserver
	./hbase-daemon.sh stop regionserver
###region无法完全启动的问题
	client.AsyncProcess: #186, table=CDC_INDEX_CDC_CASE_CASE_NAME, attempt=166/350 failed=1ops, last exception: org.apache.hadoop.hbase.NotServingRegionException: org.apache.hadoop.hbase.NotServingRegionException: Region CDC_INDEX_CDC_CASE_CASE_NAME,,1451380268609.b119e567425a4e8b94e16d931ea7ce78. is not online on hadoop151,16020,1452221378414
	关闭hbase，重启zookeeper，重启hbase即可。

###数据导入导出
	A、导出
	#for i in `./hbase shell <<< list |grep ^CDC_COMMUNITY`; do ./hbase org.apache.hadoop.hbase.mapreduce.Export $i /home/docker/HBASE_EX/$i; done
	B、导入
	#./hbase shell <<< "create 'CDC_IMP_TEST','CDC'"
	#./hbase org.apache.hadoop.hbase.mapreduce.Import CDC_IMP_TEST /home/docker/HBASE_EX/CDC_COMMUNITY_ESTATE_LDXX
###Hbase region频繁当掉的问题
	A、需要修改GC默认的为CMS，并调整GC的时机，在Hbase-ev.sh正增加如下参参数：
	export HBASE_OPTS="-server -verbose:gc -XX:NewRatio=2 -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=60 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/logs/jvm/dump -Xloggc:/logs/jvm/gc/hbasegc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps"
###记一次phoenix的性能优化：
	现象：使用phoenix查询一张表非常慢，基本查不出来；使用hbase中的count，发现会停在固定的rowkey。
	问题定位：最早以为是phoenix的问题，查询其SYSTEM.stats表，发现表中的region不全，怀疑是phoenix没有同步stast，后经过升级phoenix和后发现问题依旧，开始怀疑是Hbase的问题，使用count后，发现有卡住在某些rowkey上，打开hbase的后台发现，其中的region确实有乱序的。问题确认。
	问题处理：现在回忆起来，前几天做过一次regionmerge，merge的时候没有判断两个region是否相邻。自己写一个检测region的startkey和endkey，发现大量的不连续。只能进行人工处理了，花了一整天时间。
###记HBase的meta的问题
	#./hbase hbck的时候，同时出现多种错误
	A：ERROR:Empty REGIONINFO_QUALITIER found in hbase:meta 
		解决办法：
		hbase hbck -fixEmptyMetaCells，但是必须先解决下面的问题。
	B:ERROR: Region { meta => CDC_PARTY_SCORE,342501196702141299_342501196208086490_00000000,1482909346990.59c36efaaae34f0f961f0b0e65d57885., hdfs => hdfs://hadoop-gd10:9100/hbase/data/default/CDC_PARTY_SCORE/59c36efaaae34f0f961f0b0e65d57885, deployed => hadoop-gd51,16020,1488026120035;CDC_PARTY_SCORE,342501196702141299_342501196208086490_00000000,1482909346990.59c36efaaae34f0f961f0b0e65d57885., replicaId => 0 } listed in hbase:meta on region server hadoop-gd13,16020,1488032615812 but found on region server hadoop-gd51,16020,1488026120035
		解决办法：
		发现网上没有好的解决办法，但是后来怀疑是其他原因引起的。在日志中发现了 “No serialized HRegionInfo in ”于是通过Hbase shell在meta表中找到了线索
		meta表中，有一些数据是有ragion但是没有info:regioninfo字段，只有“info:seqnumDuringOpen”"info:server""info:serverstartcode"，于是写了一个shell将这些记录删除。删除之后，这些ERROR没有了，之后在 fixEmptyMeaCells之后问题解决。
	C:ERROR: Region { meta=>test,1m\x00\x03\x1B\x15,1393439284371.4c213a47bba83c47075f21fec7c6d862., hdfs => hdfs://master:9000/hbase/test/4c213a47bba83c47075f21fec7c6d862, deployed =>  } not deployed on any region server.
		解决办法：
		这个问题是进行人工的merge后出现的，等Hbase自己恢复，或者：
		#hbase hbck -repaire
		但是旺旺不能一次修复，需要重复执行几次,。
		也可以针对一个表repaire
		#hbase hbck -repaire 'CDC_PERSON'
		如果提示 region still in transation,waiting for become ...
		关闭hbase->删除zk中的/hbase目录->重启hbase
	D:thereis a hole in region chain between...

	E:you need to create a new .regioninfo and region dir in hdfs to plug the hole
		#hbase hbck -repair
	F:ERROR:found lingering reference file
		#hbase hbck -fixReferenceFiles
###hbase启动后，webUI上提示banlance is no enable
		#hbase shell
		#balance_switch false
		#balance_switch true
###hbase region not onlie state=failed_close
	#hbase hbck -repaire

###hbase pagefilter的结果大于pagefilter设置的值
	this filter cannot guarantee that the number of results returned to a client are <= page size. This is because the filter is applied separately on different region servers. It does however optimize the scan of individual HRegions by making sure that the page size is never exceeded locally.
	setMaxResultSize();
	G:on hdfs but not listed in hbase:meta
	rm region file on hdfs

###No serialized HRegionInfo in keyvalues={CDC_ALARM_PERSON_TEMP,....}
	
