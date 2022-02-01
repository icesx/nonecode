HDFS
===

### 注意事项	

1. 删除文件

   ```
   ./hadoop fs -rm -r -f /tmp/logs/docker/logs/
   ```

   

2. 文件系统检测

  ```
  bin/hadoop fsck / -delete
  ```


  一般不要去执行-delete，因为hadoop自己会进行分片的复制

3. hadoop启动会自动去进行分片的平衡，当删除一个datenode后，或者增加一个datenode后，hadoop会自动将丢失的分片找回来

4. slow readProcessor read fields took 。。。ms

5. There appears to be a gap in the edit log
    -recover

6. 离开safemode

   ```
   hadoop dfsadmin -safemode leave 
   ```

### 设置分片
	$hadoop dfs -setrep -w 2 -R /hbase/data/default/CDC_PARTY_SCORE
### 回复文件

```
 ./hdfs debug recoverLease -path /hbase/oldWALs/pv2-00000000000000000478.log
```



### Slow BlockReceiver write packet to mirror

可能是网线降速了，检查网线是不是变黄了