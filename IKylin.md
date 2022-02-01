Kylin
===

## 安装

1. 版本

   kylin对版本的说明比较笼统，安装中发现还是有问题的。

   经测试通过的版本为：

2. 配置

注：经测试发现kylin3.x.x智能支持spark2.x.x，不支持spark3.x.x

```sh
cat > EOF .profile <<EOF
export JAVA_HOME=/home/bjrdc/software/jdk1.8.0_131_x64
export HBASE_HOME=/home/bjrdc/software/hbase-2.2.6
export HADOOP_HOME=/home/bjrdc/software/hadoop-3.0.3
export HADOOP_CONF_DIR=/home/bjrdc/software/hadoop-3.0.3/etc/hadoop
export HIVE_HOME=/home/bjrdc/software/apache-hive-3.1.0-bin
export SPARK_HOME=/home/bjrdc/software/spark-2.4.8-bin-hadoop2.7
export PATH=$PATH:$JAVA_HOME/bin:$HBASE_HOME/bin:$HADOOP_HOME/bin:$HIVE_HOME/bin
EOF
```
2. push spark-libs.jar

   ```sh
   cd $SPARK_HOME
   jar -cvf spark-libs-2.4.8.jar -C jars/ .
   $HADOOP_HOME/bin/hdfs dfs -put spark-libs-2.4.8.jar /spark-libs/
   ```

   

3. spark.default.conf

   ```properties
   spark.yarn.archive=hdfs://bjrdc10:9100/spark-libs/spark-libs-2.4.8.jar
   ```

4. kylin.properties

   ```properties
   kylin.engine.spark-conf.spark.master=yarn
   kylin.engine.spark-conf.spark.submit.deployMode=cluster
   kylin.engine.spark-conf.spark.yarn.queue=default
   kylin.engine.spark-conf.spark.driver.memory=2G
   kylin.engine.spark-conf.spark.executor.memory=4G
   kylin.engine.spark-conf.spark.executor.instances=40
   kylin.engine.spark-conf.spark.shuffle.service.enabled=false
   kylin.engine.spark-conf.spark.eventLog.enabled=true
   kylin.engine.spark-conf.spark.yarn.archive=hdfs://bjrdc10:9100/spark-libs/spark-libs-2.4.8.jar
   ```

   

5. 启动

   ```sh
   ${kylin_home}/bin/kylin.sh start
   ```

6. 访问

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
cp $SPARK_HOME/jars/spark-core_2.11-2.4.8.jar $KYLIN_LIB
cp $SPARK_HOME/jars/scala-library-2.11.12.jar $KYLIN_LIB
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

#### class notfound org/apache/spark/sql/execution/columnar/CachedBatch

最终通过查看源代码，发现是版本不兼容，kylin3，只能支持到spark2

#### 多版本引擎的无法登录

```
${KYLIN_HOME}/bin/metastore.sh remove /user/admin
```

**注：一定放到`kylin/lib/`**目录下，放到tomcat/webapps/kylin/WEN-INFO/lib下是有问题的。

### Couldn't locate hcatalog installation, please make sure it is installed and set HCAT_HOME to the path

```sh
export HIVE_LIB=$HIVE_HOME/lib
```



### 配置Spark

### kylin.properties

```properties
kylin.engine.spark-conf.spark.submit.deployMode=cluster
kylin.engine.spark-conf.spark.shuffle.service.enabled=false
kylin.engine.spark-conf.spark.yarn.archive=hdfs://bjrdc10:9100/kylin/spark-libs/spark-libs-2.4.8.jar
```

#### spark-libs.jar

```sh
jar cv0f spark-libs.jar -C $SPARK_HOME/jars/ ./
```



## 使用

### 样本

```sql
select part_dt, sum(price) as total_sold, count(distinct seller_id) as sellers from kylin_sales group by part_dt order by part_dt
```

![image-20210817164706434](/ICESX/ISunflower/nonecode/IKylin.assets/image-20210817164706434.png)

```sql
SELECT SUM(price) AS gmv
 FROM kylin_sales 
INNER JOIN kylin_cal_dt AS kylin_cal_dt
 ON kylin_sales.part_dt = kylin_cal_dt.cal_dt
 INNER JOIN kylin_category_groupings
 ON kylin_sales.leaf_categ_id = kylin_category_groupings.leaf_categ_id AND kylin_sales.lstg_site_id = kylin_category_groupings.site_id
 WHERE kylin_cal_dt.cal_dt between DATE '2013-09-01' AND DATE '2013-10-01' AND (lstg_format_name='FP-GTC' OR 'a' = 'b')
 GROUP BY kylin_cal_dt.cal_dt;

```



```sql
 
SELECT kylin_sales.part_dt, seller_id
FROM kylin_sales
INNER JOIN kylin_cal_dt AS kylin_cal_dt
ON kylin_sales.part_dt = kylin_cal_dt.cal_dt
INNER JOIN kylin_category_groupings
ON kylin_sales.leaf_categ_id = kylin_category_groupings.leaf_categ_id
AND kylin_sales.lstg_site_id = kylin_category_groupings.site_id 
GROUP BY 
kylin_sales.part_dt, kylin_sales.seller_id ORDER BY SUM(kylin_sales.price) DESC LIMIT 20;
```

