Kylin
===

## 安装

1. 版本

   kylin对版本的说明比较笼统，安装中发现还是有问题的。

   经测试通过的版本为：

2. 配置

```sh
kylin=~/software/apache-kylin-3.1.2-bin-hadoop3/bin/header.sh
echo HBASE_HOME=~/software/hbase-2.2.6 >> $kylin
echo HADOOP_HOME=~/software/hadoop-3.0.3 >> $kylin
echo HIVE_HOME=~/software/apache-hive-3.1.0-bin >> $kylin
echo SPARK_HOME=/home/bjrdc/software/spark-3.1.2-bin-hadoop3.2 >> $kylin
echo export PATH=$PATH:"$HBASE_HOME"/bin:"$HADOOP_HOME"/bin:"$HIVE_HOME"/bin >> $kylin
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
### 问题处理

#### 启动的时候缺少jar包

```sh
KYLIN_LIB=~/software/apache-kylin-3.0.1-bin-hadoop3/lib/
HIVE_LIB=~/software/apache-hive-3.1.0-bin/lib/
cp $HIVE_LIB/hive-standalone-metastore-*.jar $KYLIN_LIB
cp $HIVE_LIB/hive-storage-api-*.jar $KYLIN_LIB
cp $HIVE_LIB/hive-common-*jar $KYLIN_LIB
cp $HIVE_LIB/libfb303-*.jar $KYLIN_LIB
cp $HIVE_LIB/jdo-api-*.jar $KYLIN_LIB                          
cp $HIVE_LIB/javax.jdo-*.jar $KYLIN_LIB
cp $HIVE_LIB/datanucleus*.jar $KYLIN_LIB
cp $HIVE_LIB/mysql-connector-java-5.1.32.jar $KYLIN_LIB
cp $HIVE_LIB/hive-exec-*.jar $KYLIN_LIB
cp $HIVE_LIB/antlr-runtime-*.jar $KYLIN_LIB 
cp $HIVE_LIB/hive-hcatalog-core-*.jar $KYLIN_LIB                         
```

详细的的jar对应关系如下：

#### Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.hive.common.ValidWriteIdList

```sh
cp $HIVE_LIB/hive-storage-api-*.jar $KYLIN_LIB
```

#### Caused by: java.lang.ClassNotFoundException: com.facebook.fb303.FacebookService$Iface

```
cp $HIVE_LIB/libfb303-*.jar $KYLIN_LIB
```



#### 多版本引擎的无法登录

```
${KYLIN_HOME}/bin/metastore.sh remove /user/admin
```

**注：一定放到`kylin/lib/`**目录下，放到tomcat/webapps/kylin/WEN-INFO/lib下是有问题的。

## 使用

### 样本

```sql
select part_dt, sum(price) as total_sold, count(distinct seller_id) as sellers from kylin_sales group by part_dt order by part_dt
```

![image-20210817164706434](/ICESX/ISunflower/nonecode/IKylin.assets/image-20210817164706434.png)

