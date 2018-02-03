1、build
	修改hbase的版本为1.1.2
	mvn -Phadoop_2 -fae -DskipTests clean package
2、config hadoop 
	mapreduce.jobtracker.address=hadoop-cx11
3、 
