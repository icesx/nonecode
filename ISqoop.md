1、安装
	A、下载最新版本“sqoop-1.99.7-bin-hadoop200.tar.gz”并解压
	B、$sqoop_home/bin目录下
		I、增加sqoop-env.sh
			export HADOOP_HOME=/home/docker/software/hadoop-2.6.0
		II、vi sqoop.sh
			source ./sqoop-env.sh
	C、$sqoop_home/conf
		vi sqoop.properties
		org.apache.sqoop.submission.engine.mapreduce.configuration.direcotry=/home/docker/software/hadoop-2.6.0/etc/hadoop
	D、$sqoop_home/bin/sqoop2-tool verify
	E、$sqoop_home/bin/sqoop2-tool upgrade
	注：如上两个命令需要在启动服务之前
	F、启动sqoop2-server
		#sqoop2-server.sh start
2、使用
	A、set server --host hadoop-xj175
	B、

3、开发
	A、异常
		org.apache.sqoop.common.SqoopException: CLIENT_0004:Unable to find valid Kerberos ticket cache (kinit)
		这个可能是服务器没有起来，使用浏览器输入地址：http://hadoop-xj175:12000/sqoop/version,如果输出正常就说明服务启动正常，否则看看是否安装成功

4、sqoop-1.99.6版本的安装和1.99.7不同
	需要在/hadoop/sqoop-1.99.6/server/conf/catalina.properties 把/home/docker/software/hadoop-2.6.0/share/hadoop下所有*.jar包加入到类路径里.
	类似：
		common.loader=${catalina.base}/lib,${catalina.base}/lib/*.jar,${catalina.home}/lib,${catalina.home}/lib/*.jar,${catalina.home}/../lib/*.jar,/home/docker/software/hadoop-2.6.0/share/hadoop/hdfs/*.jar,/home/docker/software/hadoop-2.6.0/share/hadoop/hdfs/lib/*.jar,/home/docker/software/hadoop-2.6.0/share/hadoop/mapreduce/*.jar,/home/docker/software/hadoop-2.6.0/share/hadoop/mapreduce/lib/*.jar,/home/docker/software/hadoop-2.6.0/share/hadoop/yarn/*.jar,/home/docker/software/hadoop-2.6.0/share/hadoop/yarn/lib/*.jar,/usr/lib/hive/lib/*.jar,/home/docker/software/hadoop-2.6.0/share/hadoop/common/*.jar,/home/docker/software/hadoop-2.6.0/share/hadoop/common/lib/*.jar
