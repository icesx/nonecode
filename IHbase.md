HBase
====
## install

### zookeeper安装
	A、配置环境变量
			export JAVA_HOME=/home/docker/software/jdk1.7.0_71_linux_x64
	B、配置zoo.cfg
		dataDir=/home/docker/volume/ZOOKEEPER
		clientPort=2181
		server.1=172.16.1.7:2888:3888
		server.2=172.16.1.8:2888:3888
		server.3=172.16.1.107:2888:3888
	C、创建myid文件
		在server.1上执行如下命令		
		echo 1 >/home/docker/volume/ZOOKEEPER/myid
	D、启动zookeeper
		bin/zkServer.sh start
### Hbase安装
```sh
A、配置 JAVA_HOME
	hbase_env.sh中增加
		export JAVA_HOME=/home/docker/software/jdk1.7.0_71_linux_x64
		export HBASE_MANAGES_ZK=false
B、配置hbase-site.xml
```

vi /bin/hbase

```
# Add libs to CLASSPATH
for f in $HBASE_HOME/lib/*.jar; do
  CLASSPATH=${CLASSPATH}:$f;
done
```



### 问题处理

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
		```#./hbase hbck -repair```
		如果不行的话，再查看master的日志。如果出现：java.io.IOException: error or interrupted while splitting logs in。这样的异常，那么就可能是异常关集群引起的，修复办法是：
		```#./hadoop fs -rm -r /hbase/WALs/hadoop-xj166,16020,1472551606664-splitting```

### 在进行Hbase的的版本切换的时候，要删除Hadoop Hdfs中的hbase目录中的文件，否则出现版本兼容问题。



### Hadoop中增加Hbase的环境变量

```
	vi hadoop/etc/hadoop/hadoop-env.sh
	export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:`/home/docker/software/hbase-1.1.2/bin/hbase classpath`	
```



## Hbase shell



```sql
create  'xjgz_portal',{NAME=>'portal'}
put 'PHOENIX_TEST',"\x80\x00\x00\x00",'0:newcolumn','nnnnneeeeewwwww'
get 'PHOENIX_TEST',"\x80\x00\x00\x00"	【"\x80\x00\x00\x00" 为rowkey】
scan 'PHOENIX_TEST',{LIMIT=>5}
create 'KeyPerson','CDC'
put 'KeyPerson','610123190011234624','CDC:name','zhangzongchang'
list
scan 'hbase:meta',{STARTROW=>'COMPONENT.PHOENIX_TEST',LIMIT=>2}
deleteall 'hbase:meta','COMPONENT.PHOENIX_TEST'
```


### 单独启动region

	./hbase-daemon.sh start regionserver
	./hbase-daemon.sh stop regionserver
### region无法完全启动的问题
	client.AsyncProcess: #186, table=CDC_INDEX_CDC_CASE_CASE_NAME, attempt=166/350 failed=1ops, last exception: org.apache.hadoop.hbase.NotServingRegionException: org.apache.hadoop.hbase.NotServingRegionException: Region CDC_INDEX_CDC_CASE_CASE_NAME,,1451380268609.b119e567425a4e8b94e16d931ea7ce78. is not online on hadoop151,16020,1452221378414
	关闭hbase，重启zookeeper，重启hbase即可。

### 数据导入导出
	A、导出
	#for i in `./hbase shell <<< list |grep ^CDC_COMMUNITY`; do ./hbase org.apache.hadoop.hbase.mapreduce.Export $i /home/docker/HBASE_EX/$i; done
	B、导入
	#./hbase shell <<< "create 'CDC_IMP_TEST','CDC'"
	#./hbase org.apache.hadoop.hbase.mapreduce.Import CDC_IMP_TEST /home/docker/HBASE_EX/CDC_COMMUNITY_ESTATE_LDXX
### Hbase region频繁当掉的问题
	A、需要修改GC默认的为CMS，并调整GC的时机，在Hbase-ev.sh正增加如下参参数：
	export HBASE_OPTS="-server -verbose:gc -XX:NewRatio=2 -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=60 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/logs/jvm/dump -Xloggc:/logs/jvm/gc/hbasegc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps"
### 记一次phoenix的性能优化：
	现象：使用phoenix查询一张表非常慢，基本查不出来；使用hbase中的count，发现会停在固定的rowkey。
	问题定位：最早以为是phoenix的问题，查询其SYSTEM.stats表，发现表中的region不全，怀疑是phoenix没有同步stast，后经过升级phoenix和后发现问题依旧，开始怀疑是Hbase的问题，使用count后，发现有卡住在某些rowkey上，打开hbase的后台发现，其中的region确实有乱序的。问题确认。
	问题处理：现在回忆起来，前几天做过一次regionmerge，merge的时候没有判断两个region是否相邻。自己写一个检测region的startkey和endkey，发现大量的不连续。只能进行人工处理了，花了一整天时间。
### 记HBase的meta的问题
./hbase hbck的时候，同时出现多种错误
1. ERROR:Empty REGIONINFO_QUALITIER found in hbase:meta 
解决办法：
```
hbase hbck -fixEmptyMetaCells，但是必须先解决下面的问题。
```
2. ERROR: Region { meta => CDC_PARTY_SCORE,342501196702141299_342501196208086490_00000000,1482909346990.59c36efaaae34f0f961f0b0e65d57885., hdfs => hdfs://hadoop-gd10:9100/hbase/data/default/CDC_PARTY_SCORE/59c36efaaae34f0f961f0b0e65d57885, deployed => hadoop-gd51,16020,1488026120035;CDC_PARTY_SCORE,342501196702141299_342501196208086490_00000000,1482909346990.59c36efaaae34f0f961f0b0e65d57885., replicaId => 0 } listed in hbase:meta on region server hadoop-gd13,16020,1488032615812 but found on region server hadoop-gd51,16020,1488026120035
		解决办法：
		发现网上没有好的解决办法，但是后来怀疑是其他原因引起的。在日志中发现了 “No serialized HRegionInfo in ”于是通过Hbase shell在meta表中找到了线索
		meta表中，有一些数据是有ragion但是没有info:regioninfo字段，只有“info:seqnumDuringOpen”"info:server""info:serverstartcode"，于是写了一个shell将这些记录删除。删除之后，这些ERROR没有了，之后在 fixEmptyMeaCells之后问题解决。
3. ERROR: Region { meta=>test,1m\x00\x03\x1B\x15,1393439284371.4c213a47bba83c47075f21fec7c6d862., hdfs => hdfs://master:9000/hbase/test/4c213a47bba83c47075f21fec7c6d862, deployed =>  } not deployed on any region server.
解决办法：
这个问题是进行人工的merge后出现的，等Hbase自己恢复，或者：
```
hbase hbck -repaire
```
但是旺旺不能一次修复，需要重复执行几次,。也可以针对一个表repaire
```
hbase hbck -repaire 'CDC_PERSON'
```
如果提示 region still in transation,waiting for become ...
关闭hbase->删除zk中的/hbase目录->重启hbase
5. :thereis a hole in region chain between...
	
6. :you need to create a new .regioninfo and region dir in hdfs to plug the hole
```#hbase hbck -repair```
7. :ERROR:found lingering reference file
```#hbase hbck -fixReferenceFiles```
### hbase启动后，webUI上提示banlance is no enable
```
hbase shell
balance_switch false
balance_switch true
```
### hbase region not onlie state=failed_close
```
hbase hbck -repaire
```
### hbase pagefilter的结果大于pagefilter设置的值
	this filter cannot guarantee that the number of results returned to a client are <= page size. This is because the filter is applied separately on different region servers. It does however optimize the scan of individual HRegions by making sure that the page size is never exceeded locally.
	setMaxResultSize();
	G:on hdfs but not listed in hbase:meta
	rm region file on hdfs

### No serialized HRegionInfo in keyvalues={CDC_ALARM_PERSON_TEMP,....}

### org.apache.hadoop.hbase.PleaseHoldException: Master is initializing



## importTsv

### Hbase进行大量数据导入的时候，通过API或者Mapreduce的方式太慢，经过研究发现，可以使用TSV导入的方式进行，又可以分为通过PUT和直接写HFile两种方式。
1. put 方式

```
./hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator="," -Dimporttsv.columns="CDC:CREATETIME,CDC:UPDATETIME,HBASE_ROW_KEY,CDC:ZPLX,CDC:DJR,CDC:ZHXGR,CDC:ZHXGJGMC,CDC:ZHXGJGDM,CDC:DJJGDM,CDC:RID,CDC:RKRQ,CDC:BZ,CDC:DJJGMC,CDC:ZHXGJG,CDC:ZHXGRXM,CDC:ZHXGSJ,CDC:DJJG,CDC:ETL_DATE,CDC:YXZT,CDC:ID,CDC:DJRDM,CDC:DJRXM,CDC:ZHXGRDM,CDC:DJSJ,CDC:YXYCKZPXSF,CDC:CARDNO,CDC:ZPXX,CDC:ZPXXBASE64" CDC_F_PSBW_P_RK_ZPXX hdfs://hadoop-gd10:9100/xjgz/cdc/import_csv/CDC_F_PSBW_P_RK_ZPXX.json.csv.csv 
$bin/hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns=a,b,c <tablename> <hdfs-inputdir>
```

2. 创建hfile

```
	./hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator="," -Dimporttsv.columns="CDC:CREATETIME,CDC:UPDATETIME,HBASE_ROW_KEY,CDC:ZPLX,CDC:DJR,CDC:ZHXGR,CDC:ZHXGJGMC,CDC:ZHXGJGDM,CDC:DJJGDM,CDC:RID,CDC:RKRQ,CDC:BZ,CDC:DJJGMC,CDC:ZHXGJG,CDC:ZHXGRXM,CDC:ZHXGSJ,CDC:DJJG,CDC:ETL_DATE,CDC:YXZT,CDC:ID,CDC:DJRDM,CDC:DJRXM,CDC:ZHXGRDM,CDC:DJSJ,CDC:YXYCKZPXSF,CDC:CARDNO,CDC:ZPXX,CDC:ZPXXBASE64" -Dimporttsv.bulk.output=hdfs://hadoop-gd10:9100/xjgz/cdc/import_csv/CDC_F_PSBW_P_RK_ZPXX CDC_F_PSBW_P_RK_ZPXX hdfs://hadoop-gd10:9100/xjgz/cdc/import_csv/CDC_F_PSBW_P_RK_ZPXX.json.csv.csv 
```
```
$ bin/hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns=a,b,c -Dimporttsv.bulk.output=hdfs://storefile-outputdir <tablename> <hdfs-data-inputdir>
[这种方式可以判断是否格式的问题，有问题的话，控制台会提示]
```
3. 发现使用入行命令创建的时候，mapreduce并没有进入到hadoop的管理界面上。并且产生的临时文件存储在了/tmp目录下，即使修改了hadoop和hbase的配置也没有用。故改为使用hadoop的方式执行mapreduce，命令如下
```
./hadoop jar ../../hbase-1.1.2/lib/hbase-server-1.1.2-jar importtst -Dimporttsv.separator="," -Dimporttsv.columns="CDC:CREATETIME,CDC:UPDATETIME,HBASE_ROW_KEY,CDC:ZPLX,CDC:DJR,CDC:ZHXGR,CDC:ZHXGJGMC,CDC:ZHXGJGDM,CDC:DJJGDM,CDC:RID,CDC:RKRQ,CDC:BZ,CDC:DJJGMC,CDC:ZHXGJG,CDC:ZHXGRXM,CDC:ZHXGSJ,CDC:DJJG,CDC:ETL_DATE,CDC:YXZT,CDC:ID,CDC:DJRDM,CDC:DJRXM,CDC:ZHXGRDM,CDC:DJSJ,CDC:YXYCKZPXSF,CDC:CARDNO,CDC:ZPXX,CDC:ZPXXBASE64" -Dimporttsv.bulk.output=hdfs://hadoop-gd10:9100/xjgz/cdc/import_csv/CDC_F_PSBW_P_RK_ZPXX CDC_F_PSBW_P_RK_ZPXX hdfs://hadoop-gd10:9100/xjgz/cdc/import_csv/CDC_F_PSBW_P_RK_ZPXX.json.csv.csv
```
4. 加载

   ```	
   ./hadoop jar /home/docker/software/hbase-1.1.2/lib/hbase-server-1.1.2.jar completebulkload   hdfs://hadoop-gd10:9100/xjgz/cdc/bulk/CDC_F_PSBW_P_RK_ZPXX/ CDC_F_PSBW_P_RK_ZPXX
   	注：加载后发现Hfile并不是256MB一个，可能很大，如上的例子中创建的HFile有10GB大小，估计Hbase后后续自己进行分割吧[经验证，Hbase会自己做平衡的]
   ```

   

3. 查看结果
	./hbase org.apache.hadoop.hbase.mapreduce.RowCounter 'tableName'



## Hbase

### 基本操作

#### 常用命令

```
echo "list_locks"| hbase shell &> /tmp/locks.txt
echo "list_procedures"| hbase shell &> /tmp/procedures.txt
```



## HBCK2

https://github.com/apache/hbase-operator-tools/tree/master/hbase-hbck2

### 安装

#### 编译jar

官方安装说明为`maven install`方式，需要将代码拿下来后，自行编译，编译好的jar文件复制到hbase/lib目录下即可。

#### 配置环境变量

```
cat conf/hbase-env.sh |grep HADOOP_HOME
export HADOOP_HOME=/home/bjrdc/software/hadoop-3.0.3
```



### 基本命令

#### help

HBCK2 help包含了大量帮助信息

```sh
./hbase org.apache.hbase.HBCK2  --help
```

#### addFsRegionsMissingInMeta

可以将文件系统中有的，meta中没有的regions信息建到meta中，`default`为namespace

```
./hbase org.apache.hbase.HBCK2   addFsRegionsMissingInMeta default
```

执行成功后会提示`assigns`regions

```
Regions re-added into Meta: 43
WARNING: 
        43 regions were added to META, but these are not yet on Masters cache. 
You need to restart Masters, then run hbck2 'assigns' command below:
                assigns a4900f31fd9e686f9b2de8c394a20aed cce4ec2af61ae6babc44bf3cc22bc240 326b0e1a84064e9689343fbcf237c839 0db98650d97769e29c82d23997730bec 
```

然后使用assigns命令进行操作

#### assigns

分配region，可以包含多个region。

```
./hbase org.apache.hbase.HBCK2  assigns a4900f31fd9e686f9b2de8c394a20aed cce4ec2af61ae6babc44bf3cc22bc240 326b0e1a84064e9689343fbcf237c839 0db98650d97769e29c82d23997730bec
```



 #### bypass
   Options:
    -o,--override   override if procedure is running/stuck
    -r,--recursive  bypass parent and its children. SLOW! EXPENSIVE!
    -w,--lockWait   milliseconds to wait before giving up; default=1
   Pass one (or more) procedure 'pid's to skip to procedure finish. Parent
   of bypassed procedure will also be skipped to the finish. Entities will
   be left in an inconsistent state and will require manual fixup. May
   need Master restart to clear locks still held. Bypass fails if
   procedure has children. Add 'recursive' if all you have is a parent pid
   to finish parent and children. This is SLOW, and dangerous so use
   selectively. Does not always work.

 #### extraRegionsInMeta
   Options:
    -f, --fix    fix meta by removing all extra regions found.
   Reports regions present on hbase:meta, but with no related
   directories on the file system. Needs hbase:meta to be online.
   For each table name passed as parameter, performs diff
   between regions available in hbase:meta and region dirs on the given
   file system. Extra regions would get deleted from Meta
   if passed the --fix option.
   NOTE: Before deciding on use the "--fix" option, it's worth check if
   reported extra regions are overlapping with existing valid regions.
   If so, then "extraRegionsInMeta --fix" is indeed the optimal solution.
   Otherwise, "assigns" command is the simpler solution, as it recreates
   regions dirs in the filesystem, if not existing.
   An example triggering extra regions report for tables 'table_1'
   and 'table_2', under default namespace:

#### filesystem
   Options:
    -f, --fix    sideline corrupt hfiles, bad links, and references.
   Report on corrupt hfiles, references, broken links, and integrity.
   Pass '--fix' to sideline corrupt files and links. '--fix' does NOT
   fix integrity issues; i.e. 'holes' or 'orphan' regions. Pass one or
   more tablenames to narrow checkup. Default checks all tables and
   restores 'hbase.version' if missing. Interacts with the filesystem
   only! Modified regions need to be reopened to pick-up changes.

#### fixMeta
   Do a server-side fix of bad or inconsistent state in hbase:meta.
   Available in hbase 2.2.1/2.1.6 or newer versions. Master UI has
   matching, new 'HBCK Report' tab that dumps reports generated by
   most recent run of _catalogjanitor_ and a new 'HBCK Chore'. It
   is critical that hbase:meta first be made healthy before making
   any other repairs. Fixes 'holes', 'overlaps', etc., creating
   (empty) region directories in HDFS to match regions added to
   hbase:meta. Command is NOT the same as the old _hbck1_ command
   named similarily. Works against the reports generated by the last
   catalog_janitor and hbck chore runs. If nothing to fix, run is a
   noop. Otherwise, if 'HBCK Report' UI reports problems, a run of
   fixMeta will clear up hbase:meta issues. See 'HBase HBCK' UI
   for how to generate new execute.
   SEE ALSO: reportMissingRegionsInMeta

 #### replication
   Options:
    -f, --fix    fix any replication issues found.
   Looks for undeleted replication queues and deletes them if passed the
   '--fix' option. Pass a table name to check for replication barrier and
   purge if '--fix'.

 reportMissingRegionsInMeta <NAMESPACE|NAMESPACE:TABLENAME>...
   To be used when regions missing from hbase:meta but directories
   are present still in HDFS. Can happen if user has run _hbck1_
   'OfflineMetaRepair' against an hbase-2.x cluster. This is a CHECK only
   method, designed for reporting purposes and doesn't perform any
   fixes, providing a view of which regions (if any) would get re-added
   to hbase:meta, grouped by respective table/namespace. To effectively
   re-add regions in meta, run addFsRegionsMissingInMeta.
   This command needs hbase:meta to be online. For each namespace/table
   passed as parameter, it performs a diff between regions available in
   hbase:meta against existing regions dirs on HDFS. Region dirs with no
   matches are printed grouped under its related table name. Tables with
   no missing regions will show a 'no missing regions' message. If no
   namespace or table is specified, it will verify all existing regions.
   It accepts a combination of multiple namespace and tables. Table names
   should include the namespace portion, even for tables in the default
   namespace, otherwise it will assume as a namespace value.
   An example triggering missing regions execute for tables 'table_1'
   and 'table_2', under default namespace:

#### setRegionState 
   Possible region states:
    OFFLINE, OPENING, OPEN, CLOSING, CLOSED, SPLITTING, SPLIT,
    FAILED_OPEN, FAILED_CLOSE, MERGING, MERGED, SPLITTING_NEW,
    MERGING_NEW, ABNORMALLY_CLOSED
   WARNING: This is a very risky option intended for use as last resort.
   Example scenarios include unassigns/assigns that can't move forward
   because region is in an inconsistent state in 'hbase:meta'. For
   example, the 'unassigns' command can only proceed if passed a region
   in one of the following states: SPLITTING|SPLIT|MERGING|OPEN|CLOSING
   Before manually setting a region state with this command, please
   certify that this region is not being handled by a running procedure,
   such as 'assign' or 'split'. You can get a view of running procedures
   in the hbase shell using the 'list_procedures' command. An example
   setting region 'de00010733901a05f5a2a3a382e27dd4' to CLOSING:
     $ HBCK2 setRegionState de00010733901a05f5a2a3a382e27dd4 CLOSING
   Returns "0" if region state changed and "1" otherwise.

#### setTableState <TABLENAME> <STATE>
   Possible table states: ENABLED, DISABLED, DISABLING, ENABLING
   To read current table state, in the hbase shell run:
     hbase> get 'hbase:meta', '<TABLENAME>', 'table:state'
   A value of \x08\x00 == ENABLED, \x08\x01 == DISABLED, etc.
   Can also run a 'describe "<TABLENAME>"' at the shell prompt.
   An example making table name 'user' ENABLED:
     $ HBCK2 setTableState users ENABLED
   Returns whatever the previous table state was.

#### scheduleRecoveries <SERVERNAME>...
   Schedule ServerCrashProcedure(SCP) for list of RegionServers. Format
   server name as '<HOSTNAME>,<PORT>,<STARTCODE>' (See HBase UI/logs).
   Example using RegionServer 'a.example.org,29100,1540348649479':
     $ HBCK2 scheduleRecoveries a.example.org,29100,1540348649479
   Returns the pid(s) of the created ServerCrashProcedure(s) or -1 if
   no procedure created (see master logs for why not).
   Command support added in hbase versions 2.0.3, 2.1.2, 2.2.0 or newer.

 unassigns <ENCODED_REGIONNAME>...
   Options:
    -o,--override  override ownership by another procedure
   A 'raw' unassign that can be used even during Master initialization
   (if the -skip flag is specified). Skirts Coprocessors. Pass one or
   more encoded region names. 1588230740 is the hard-coded name for the
   hbase:meta region and de00010733901a05f5a2a3a382e27dd4 is an example
   of what a userspace encoded region name looks like. For example:
     $ HBCK2 unassigns 1588230740 de00010733901a05f5a2a3a382e27dd4
   Returns the pid(s) of the created UnassignProcedure(s) or -1 if none.

   SEE ALSO, org.apache.hbase.hbck1.OfflineMetaRepair, the offline
   hbase:meta tool. See the HBCK2 README for how to use.

### 添加丢失的meta

If hbase:meta corruption is not too critical, hbase would still be able to bring it online. Even if namespace region is among the missing regions, it will still be possible to scan hbase:meta during the initialization period, where Master will be waiting for namespace to be assigned. To verify this situation, a hbase:meta scan command can be executed as below. If it does not time out or show any errors, *hbase:meta* is online:

```
echo "scan 'hbase:meta', {COLUMN=>'info:regioninfo'}" | hbase shell
```

*HBCK2* *addFsRegionsMissingInMeta* can be used if the above does not show any errors. It reads region metadata info available on the FS region directories in order to recreate regions in hbase:meta. Since it can run with hbase partially operational, it attempts to disable online tables that are affected by the reported problem and it is going to readd regions to *hbase:meta*. It can check for specific tables/namespaces, or all tables from all namespaces. An example below shows adding missing regions for tables 'tbl_1' in the default namespace, 'tbl_2' in namespace 'n1', and for all tables from namespace 'n2':

```
$ HBCK2 addFsRegionsMissingInMeta default:tbl_1 n1:tbl_2 n2
```

### HBCK2 基本命令

hbase2 已经不支持hbck了，需要下载hbck2，并将jar放到hbase/lib/下

`https://github.com/apache/hbase-operator-tools/tree/master/hbase-hbck2`

### 

可以使用如下命令

```
./hbase org.apache.hbase.HBCK2   -h
./hbase org.apache.hbase.HBCK2   fixMeta
./hbase org.apache.hbase.HBCK2   setTableState COMPONENT.PHOENIX_TEST ENABLED
./hbase org.apache.hbase.HBCK2   addFsRegionsMissingInMeta BDP.BDP_COMPRESS_IMG
./hbase org.apache.hbase.HBCK2  assigns  e555e672cb58d3f4e16600d60f6e9f0b
./hbase org.apache.hbase.HBCK2  filesystem -f
```



```

```



### 一次恢复表的过程

参考地址https://stackoverflow.com/questions/22098754/inconsistency-in-hbase-tableregion-not-deployed-on-any-region-server

```
./hbase hbck |grep "ERROR: Region"|awk '{print $6}'
SYSTEM.LOG,\x1C\x00\x00\x00,1585395688453.f896901738f12fc5c8077f24a18e0ce2.,
SYSTEM.LOG,\x0A\x00\x00\x00,1585395688453.67800946e1f4a2a20c7b6e17be7cafb8.,
SYSTEM.LOG,\x1D\x00\x00\x00,1585395688453.669a4ac1231d6ddcfa24dc074ab2d62f.,
SYSTEM.SEQUENCE,,1585395685488.cce4ec2af61ae6babc44bf3cc22bc240.,
SYSTEM.LOG,\x03\x00\x00\x00,1585395688453.cfba075adedff23600b889fbc7d5afac.,
SYSTEM.LOG,\x07\x00\x00\x00,1585395688453.3de27b7be3c9b57ec84ad703e7be88ba.,
SYSTEM.CATALOG,,1585395681073.a4900f31fd9e686f9b2de8c394a20aed.,
SYSTEM.LOG,\x04\x00\x00\x00,1585395688453.ddd56dc28486bd1a3b94459992d01c37.,
SYSTEM.LOG,\x13\x00\x00\x00,1585395688453.71afc4f10fd8da7e954588e4e3d04154.,
```

```
hbase shell
assign '71afc4f10fd8da7e954588e4e3d04154'
```



### 一次恢复meta的过程

> 在搞协处理的时候，将hbase搞的无法启动了，后来就手动的将phoenix_test表删除，并将`SYSTEM.CATALOG`也删除了，导致整个phoenix不能用了。
>
> 由于安装的是hbase2.0.2导致`hbase hbck`命令不明用了。后来查找资料后发现是需要用HBCK2
>
> 于是作如下操作

1. 将hdfs上的/hbase目录mv到/hbase-back

   ```
   hadoop-3.0.3/bin/hdfs dfs -mv /hbase /hbase-back
   ```

2. 升级hbase到hbase2.2.6

   在hbase-site.xml中增加，“不确定是否有效”

   ```xml
   <property>
               <name>hbase.assignment.skip.empty.regions</name>
                   <value>false</value>
           </property>
   </configuration>
   ```

   

3. 下载并编译hbck2，并将jar放到hbase2.2.6/lib下

4. 启动hbse2.2.6后验证环境ok

5. 将/hbase-back下的文件mv到hbase下

   ```
   hadoop-3.0.3/bin/hdfs dfs -mv /hbase-back/default/* /hbase/default/
   ```

6. 启动重启hbase后发现所有的增加的表没有region

   日志中有如下错误

   ```
   org.apache.hadoop.hbase.NotServingRegionException: hbase:namespace,1558205786137.40562c48c9210c06813adce48773cb6a. is not online on node1,16020,1596957741742
   ```

7. 在hbck2的官方查询到相关的处理办法：

   ```
   ./stop-hbase.sh 
   ./hbase org.apache.hbase.hbck1.OfflineMetaRepair
   ./start-hbase.sh 
   ./hbase org.apache.hbase.HBCK2 assigns  e555e672cb58d3f4e16600d60f6e9f0b
   ./stop-hbase.sh 
   ./start-hbase.sh 
   ```

8. 将所有的table enable即可

## 数据备份与恢复

#### name 备份

```sh
cat name_backup.sh 
set -x
workdir=/moa/backups/
rmfile=`ls -t $workdir|grep _backup.tar|tail -1`
if [ ! -n "$rmfile" ];then
        echo "empty"
else
        echo $workdir$rmfile
        rm -f $workdir$rmfile
fi
tar -czvf $workdir/`date "+%Y_%m_%d_%H_%M_%S"`_name_backup.tar.gz /moa/HDFS-YARN/name
```

