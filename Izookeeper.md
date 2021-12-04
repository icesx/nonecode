### bin/zkEnv.sh中增加JAVA_HOME环境变量

### 需要配置myid文件
		 echo 2 > /cloud/ZOOKEEPER-SOLR/myid

### 版本问题：hbase1.x使用zookeeper3.4.X

### zkcli.sh -server hadoop-xj118:2181
	
### zk server端timeout参数，客户端超时时间实际限制在[2*tickeTime, 20*tickTime]范围内
	tickTime：zk的心跳间隔（heartbeat interval），也是session timeout基本单位。单位为微妙。
	minSessionTimeout: 最小超时时间，zk设置的默认值为2*tickTime。
	maxSessionTimeout：最大超时时间，zk设置的默认值为20*tickTime。
	经常会认为创建zookeeper客户端时设置了sessionTimeout为100s，而没有改变server端的配置，默认值是 不会生效的。 原因： 客户端的zookeeper实例在创建连接时，将sessionTimeout参数发送给了服务端，服务端会根据对应的  minSession/maxSession Timeout的设置，强制修改sessionTimeout参数，也就是修改为4s~40s 返回的参数。所以服务端不一定会以客户端的sessionTImeout做为session expire管理的时间。
	maxClientCnxns:
	
	
