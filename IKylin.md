Kylin
===

## 安装

1. 配置

```sh
kylin=~/software/apache-kylin-3.0.0-bin-hadoop3/bin/header.sh
echo HBASE_HOME=~/software/hbase-2.0.2 >> $kylin
echo HADOOP_HOME=~/software/hadoop-3.0.3 >> $kylin
echo HIVE_HOME=~/software/apache-hive-3.1.0-bin >> $kylin
echo SPARK_HOME=/home/bjrdc/software/spark-2.3.1-bin-hadoop2.7 >> $kylin
echo export PATH=$PATH:$HBASE_HOME/bin:$HADOOP_HOME/bin:$HIVE_HOME/bin >> $kylin
```
2. 启动

```sh
${kylin_home}/bin/kylin.sh start
```
http://kylin_host:7070/kylin
帐号：ADMIN	密码：KYLIN

3. HIVE 数据库名

```sh
echo kylin.source.hive.database-for-flat-table=CDC >> ${kylin_home}/conf/kylin.properties
```
## 使用

