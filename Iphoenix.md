###install
To install a pre-built phoenix, use these directions:

Download and expand the latest phoenix-[version]-bin.tar.
Add the phoenix-[version]-server.jar to the classpath of all HBase region server and master and remove any previous version. An easy way to do this is to copy it into the HBase lib directory (use phoenix-core-[version].jar for Phoenix 3.x)
Restart HBase.
Add the phoenix-[version]-client.jar to the classpath of any Phoenix client.

### 使用
$sqlline.py zookeeper:2181

### 
