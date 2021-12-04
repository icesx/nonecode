Kafka
===
## 安装
1. 单机安装

   解压后执行如下命令启动一个zookeeper

   ```
   ./bin/zookeeper-server-start.sh ../config/zookeeper.properties&
   ```

   启动kafka

   ```
   ./bin/kafka-server-start.sh ../config/server.properties
   ```

### 相关命令
```sh
#./bin/kafka-topics.sh --list --zookeeper localhost:2181
#./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
#./bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
#./kafka-console-producer.sh --topic site-alarm --broker-list localhost:9092
#./kafka-console-consumer.sh --topic site-alarm --zookeeper localhost
```
### 配置修改
	kafka默认是将数据文件存储在/tmp/kafka-logs
	需要修改到制定的空间比较大的文件中。修改的方式是
	vi ${kafka_home}/config/server.properties
	log.dirs=/cloud/kafka-logs
### zookeeper配置
	vi ${kafka_home}/config/zookeeper.properties
	dataDir=/cloud/zookeeper

### 异常处理
	在一次kfuka由于硬盘满了挂掉后，即使回复了kafuka，spark也没有再从kafuka中获取数据，知道重启spark的应用——原因未知
### 自启动
```sh
su - bjrdc -c "source ~/.profile"
su - bjrdc -c  "nohup ~/software/kafka_2.10-0.9.0.1/bin/zookeeper-server-start.sh ~/software/kafka_2.10-0.9.0.1/config/zookeeper.properties &"
su - bjrdc -c "~/software/kafka_2.10-0.9.0.1/bin/kafka-server-start.sh ~/software/kafka_2.10-0.9.0.1/config/server.properties &"
```

## Kafdrop

