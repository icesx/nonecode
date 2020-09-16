Hive
========
### 安装
#### mysql

```sql
GRANT  ALTER,USAGE,DROP,SELECT, INSERT, UPDATE, DELETE, CREATE,INDEX,SHOW VIEW ,CREATE TEMPORARY TABLES,EXECUTE ON hive_metastore.* TO 'hive'@'%' IDENTIFIED BY  'hive123';
```
### debug
```sh
hive --hiveconf hive.root.logger=DEBUG,console
```
### create metastor
```sh
schematool -dbType mysql -initSchema
```
### 创建原始表
```sql
Create table baseball.Master
	(lahmanID int, playerID STRING, managerID STRING, hofID STRING, birthyear INT,
	birthMonth INT, birthDay INT, birthCountry STRING, birthState STRING,
	birthCity STRING, deathYear INT, deathMonth INT, deathDay INT,
	deathCountry STRING, deathState STRING, deathCity STRING,
	nameFirst STRING, nameLast STRING, nameNote STRING, nameGive STRING,
	nameNick STRING, weight decimal, height decimal, bats STRING,
	throws STRING, debut INT, finalGame INT,
	college STRING, lahman40ID STRING, lahman45ID STRING, retroID STRING,
	holtzID STRING, hbrefID STRING )
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;
	 load data local inpath '/ICESX/ICESX/workSpaceJava/cloudbase/code/demo/lahman2012-csv/Master.csv' overwrite into table master;
```


```sql
Create table baseball.batting (playerID string, yearID int, stint int, teamID String , lgID string,G int,G_batting int ,AB int ,R int, H int ,B2B int, B3B int, HR int, RBI int, SB int,CS int,BB int,SO int,IBB int ,HBP int,SH int ,SF int,GIDP int ,G_old int) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;
 load data local inpath '/ICESX/ICESX/workSpaceJava/cloudbase/code/demo/lahman2012-csv/Batting.csv' overwrite into table batting;
```


```sql
create table baseball.pitching(playerID String,yearID int,stint int,teamID String,lgID string,W int,L int ,G int ,GS int ,CG int ,SHO int,SV int,IPouts int,H int,ER int,HR int,BB int,SO int,BAOpp int,ERA int,IBB int,WP int,HBP int,BK int,BFP int,GF int,R int,SH int,SF int,GIDP int)ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;
	 load data local inpath '/ICESX/ICESX/workSpaceJava/cloudbase/code/demo/lahman2012-csv/Pitching.csv' overwrite into table pitching;
```


```sql
create table baseball.fielding(playerID string,yearID int,stint int,teamID string,lgID string,POS string,G int,GS int,InnOuts int,PO int,A int,E int,DP int,PB int,WP int,SB int,CS int,ZR int)ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;
load data local inpath '/ICESX/ICESX/workSpaceJava/cloudbase/code/demo/lahman2012-csv/Fielding.csv' overwrite into table fielding;
```


```sql
Select A.PlayerID, B.teamID, B.AB, B.R, B.H, B.B2B, B.B3B, B.HR, B.RBI FROM Master A JOIN BATTING B ON A.playerID = B.playerID;
```
### 创建数据仓库事实表

```sql
Create Database baseball_stats;
Create table baseball_stats.fact_player_stats as SELECT B.playerID, B.yearID, B.stint, B.g, g_batting, ab, B.r, B.h, b2b, b3b, B.hr, rbi, B.sb, B.cs, B.bb, B.so, B.ibb, B.hbp, B.sh, B.sf, B.gidp, w,l, F.gs, cg, sho, sv, ipouts, er, baopp, era, p.wp,bk, bfp,gf,innouts, po, a, e, dp, pb from baseball.Batting B JOIN baseball.Pitching P ON B.playerid = P.playerID JOIN baseball.fielding F ON B.playerID = F.playerID ;
```
### 使用hive分析apche日志

```sql
create external table gehua.portal_98(ip string,frank string,user_id string,time string,time_zone string,method string,url string,code string,size string,referer string,used_seconds int,used_microseconds int) row format delimited fields terminated by ' ';
load data inpath '/user/i/gehua/portal/98' overwrite into table portal_98;
load data local inpath '/ICESX/ICESX/运营商/歌华/歌华日志/2013-04-23/portal/98' overwrite into table portal_98;
select ip, count(ip) as c from portal_98 group by ip order by c desc limit 50;
```
测试结果：在我自己的电脑上【3×hadoop+2×Hbase+2×zookeeper+hive】在歌华的日志【总2874717条记录】中查找当日访问portal最多次数的前50个ip地址，用时240S

### CSDN测试

```sql
create external table csdn (id string,password string,email string) row format delimited fields terminated by '#';
select password, count(password) as c from users group by password order by c desc limit 50;
```
测试结果：600Wcsdn账户 找到密码出现最多的前50名，用时327S

### 一些有用的case

#### 每秒请求量【TPS】
```sql
from (select substr(time,0,21) as t from xjgz.portal_98) t select t,count(t) as count group by t order by count desc limit 50
```
效果与下句相同
```sql
select ti.t,count(ti.t) as count from (select substr(time,0,21) as t from portal_98) ti group by t order by count desc limit 50
```
#### 一秒内同时访问的IP【同行】
```sql
select ti.t,ti.ip from (select substr(time,9,13) as t,ip from portal_98) ti group by ti.t,ti.ip order by ti.t desc limit 50;
```
#### 一秒内同时访问的IP
```sql
select ti.ip, collect_set(ti.t) as ct from (select substr(time,9,13) as t,ip from portal_98) ti group by ti.ip  order by ct  desc limit 50;
```
#### ip访问的时间【同行】
```sql
select collect_set(ip) as cip,ct from (select ip , collect_set(t) as ct from (select substr(time,9,12) as t,ip from portal_98) ti group by ti.ip) as c group by ct limit 50;
```


```sql
select cip,ct from(select collect_set(ip) as cip,ct from (select ip , collect_set(t) as ct from (select substr(time,9,12) as t,ip from cdc.portal_98) ti group by ti.ip) c group by c.ct) e where size(cip)>2 and size(ct)>2 limit 50;
```
#### 频繁出没

```sql
select ti.t,ti.ip,count(ti.ip) as ipc from (select substr(time,9,11) as t,ip from cdc.portal_98) ti group by ti.t,ti.ip where ipc>20 order by ipc desc;
```

以天为单位统计ip出现的次数

```sql
select ti.ip,count(ti.ip) as ipc from (select substr(time,9,11) as t,ip from cdc.portal_98) ti group by ti.ip order by ipc desc limit 50;
```

### 异常处理
​	A、load data inpath '/user/i/gehua/portal/98' overwrite into table portal_98;
​		FAILED: SemanticException [Error 10028]: Line 1:18 Path is not legal ''/xjgz/cdc/resource/hitlog/98'': Move from: hdfs://hadoop1:9100/xjgz/cdc/resource/hitlog/98 to: hdfs://172.16.1.1:9100/user/hive/warehouse/cdc.db/hitlog is not valid. Please check that values for params "default.fs.name" and "hive.metastore.warehouse.dir" do not conflict.
​		使用external table 发现可以，不知道为什么！！！

```sql
create external table cdc.hitlog(ip string,frank string,user_id string,time string,time_zone string,method string,url string,code string,size string,referer string,used_seconds int,used_microseconds int) row format delimited fields terminated by ' ' location '/xjgz/cdc/resource/hitlog';
```
### 相关命令

```
./hive --service hiveserver2
show tables
```

### 兼容性
​	目前hive1.2.1 和hbase1.2不能兼容,hive-2.0.0已经兼容hbase1.x
### 自己编译hive2.0

mvn clean install -DskipTests

	mvn clean package -DskipTests -Pdist
	发现启动有一些问题，增加了hive-site.xml后问题未解决，虽然可以跑mapreduce但是select × from table 无法查询结果出来:
```

		<configuration>
		  <property>
		    <name>hive.metastore.schema.verification</name>
		    <value>false</value>
		    <description/>
		  </property>
		<property>  
		<name>datanucleus.fixedDatastore</name>  
		<value>false</value>   
		</property> 
		<property>
		  <name>datanucleus.autoCreateSchema</name>
		  <value>true</value>
		</property>

		<property>
		  <name>datanucleus.autoCreateTables</name>
		  <value>true</value>
		</property>
	
		<property>
		  <name>datanucleus.autoCreateColumns</name>
		  <value>true</value>
		</property>
	
		</configuration>
```

### 与hbase整合

​	hive -hiveconf hive.root.logger=DEBUG,console

```
	CREATE EXTERNAL TABLE cdc.phoenix_test(mykey string, mycolumn string) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,0:mycolumn")TBLPROPERTIES("hbase.table.name" = "PHOENIX_TEST");
```
	select (*) from cdc.phoenix_test
	Exception in thread "Thread-1" java.lang.OutOfMemoryError: PermGen space
	在Hadoop的hadoop-env.sh 增加
		export HADOOP_CLIENT_OPTS="-XX:MaxPermSize=256m $HADOOP_CLIENT_OPTS"

### 与hbase整合
​	A、注意大小写，当Hbase中字段为大写的时候，在创建Hivetable的时候，一定要讲mapping字段中写上大写的。

### 问题处理
+ Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: org.apache.hadoop.hive.serde2.SerDeException: HBase row key cannot be NULL
这是因为在insert的时候，有rowkey为空，如下：
```
		 INSERT OVERWRITE TABLE cdc.CDC_ANALYSIS_APPEAR_OFTEN_GOODS_SITES
		  --/*物品经常出现的场所*/
		 select concat(mac,"_",lpad(string(size(time_d_col)),4,"0"),"_",rpad(string(rand()),16,"0")),mac,site_id,concat_ws(",",time_d_col)
```
当mac字段为空或者null的时候，就出现上面的问题，解决办法是where mac<>''

+ hive> drop table cdc_analysis_walkingwith_pre;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:For direct MetaStore DB connections, we don't support retries at the client level.)

替换mysql的jar from mysql-connector-java-5.1.1.jar mysql-connector-java-5.1.32.jar
	
+ hbase Attempt to do update or delete using transaction manager that does not support these operations
https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#Configuration
注：但是惊奇的发现，如果加上如下配置后，INSERT INTO TABLE cdc.CDC_NETWORK_AUDIT_CLEAN 就无法用了——hive2.0.0】
```
		hive-site.xml增加如下配置，重启hive
		   <property>
		    <name>hive.support.concurrency</name>
		    <value>true</value>
		  </property>
		    <property>
		    <name>hive.exec.dynamic.partition.mode</name>
		    <value>nonstrict</value>
		  </property>
		  <property>
		    <name>hive.txn.manager</name>
		    <value>org.apache.hadoop.hive.ql.lockmgr.DbTxnManager</value>
		  </property>
		    <property>
		    <name>hive.compactor.initiator.on</name>
		    <value>true</value>
		  </property>
		  <property>
		    <name>hive.compactor.worker.threads</name>
		    <value>1</value>
		  </property>
```
+ Caused by: java.io.IOException: HBase row key cannot be NULL
+ Hbase 的overwrite
	Overwrite
	Another difference to note between HBase tables and other Hive tables is that when INSERT OVERWRITE is used, existing rows are not deleted from the table. However, existing rows are overwritten if they have keys which match new rows.

+ NULL值判断
	A、如果是hbase的话，可以使用如下sql来判断是否有NULL
	select * from cdc_network_audit where mac is NULL;
	但是select count(*) from cdc_network_audit where mac is NULL;没有查出来数据，是不是bug？
### 方法
```
select visit_time,regexp_replace(visit_time, '-([0-9]{2})-', '-06-') from cdc.cdc_community_mj_access limit 10;
```
### 强制mapreduce
```
	set hive.fetch.task.conversion=none[all mapreduce]
	set hive.fetch.task.conversion=more[no mapreduce]
```
