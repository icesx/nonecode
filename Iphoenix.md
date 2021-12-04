## install

### hbase1.x

To install a pre-built phoenix, use these directions:

Download and expand the latest phoenix-[version]-bin.tar.
Add the phoenix-[version]-server.jar to the classpath of all HBase region server and master and remove any previous version. An easy way to do this is to copy it into the HBase lib directory (use phoenix-core-[version].jar for Phoenix 3.x)
Restart HBase.
Add the phoenix-[version]-client.jar to the classpath of any Phoenix client.

### hbase2.x

hbase2.0.x后phoenix已经有对应的版本也有支持 [5.0.0-Hbase-2.0](http://www.apache.org/dyn/closer.lua/phoenix/apache-phoenix-5.0.0-HBase-2.0/bin/apache-phoenix-5.0.0-HBase-2.0-bin.tar.gz)

安装的时候，按照hbase1.0的安装方法后启动会报一个错误

```
java.lang.NoClassDefFoundError: org/apache/htrace/SpanReceiver
```

因为phoenix5.0.0用到了htrace，但是hbase2.0的版本又没有提供对应的版本故解决办法是下载`htrace-core-3.2.0-incubating.jar`复制到hbase的 `/home/bjrdc/software/hbase-2.3.4/lib/`目录下即可

## 使用

### 链接服务器
```
$sqlline.py zookeeper:2181
```

### 基本语法



