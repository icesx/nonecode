Ambari
======

## install

### 编译

> 默认的ambari中配置的hadoop、hbase等都是hourtonworks的，编译的过程中会下载不到包。
>
> 编译安装过程比较麻烦。

### 安装环境

1.  ambari

   `/home/bjrdc/tools/apache-ambari-2.7.5-src`

2. java

   `/home/bjrdc/software/jdk1.8.0_131_x64`

3. maven

   `/home/bjrdc/software/apache-maven-3.6.0`

4. 依赖库

   如果不安装python库中间会出现编译的错误

5. 环境变量

   `/home/bjrdc/software/jdk1.8.0_131_x64/bin:/home/bjrdc/software/apache-maven-3.6.0/bin`

### 尝试编译

1. 安装python

   ```
   apt install python-psutil pyhton-dev
   ```

2. 安装git

   ```
   apt install git
   ```

   

3. 安装nodejs

   ```sh
   sudo npm install n -g
   sudo npm --version
   npm config set registry https://registry.npm.taobao.org
   npm config ls
   ```

4. 编译

   ```sh
   wget https://www-eu.apache.org/dist/ambari/ambari-2.7.5/apache-ambari-2.7.5-src.tar.gz (use the suggested mirror from above)
   tar xfvz apache-ambari-2.7.5-src.tar.gz
   cd apache-ambari-2.7.5-src
   mvn versions:set -DnewVersion=2.7.5.0.0
    
   pushd ambari-metrics
   mvn versions:set -DnewVersion=2.7.5.0.0
   popd
   mvn versions:set -DnewVersion=2.7.5.0.0
   popd ambari-metrics
   mvn -B clean install jdeb:jdeb -DnewVersion=2.7.5.0.0 -DbuildNumber=5895e4ed6b30a2da8a90fee2403b6cab91d19972 -DskipTests -Dpython.ver="python >= 2.6" -DskipTests -Drat.skip=true
   ```

5. 安装

   ```sh
   sudo dpkg -i ambari-server_2.7.5.0-0-dist.deb 
   ```

6. 配置

   ```
   sudo ambari-server setup
   ```

   正常情况下有如下输出

   ```
   Enter choice (1): 1
   Database admin user (postgres): 
   Database name (ambari): 
   Postgres schema (ambari): 
   Username (ambari): 
   Enter Database Password (bigdata): 
   Default properties detected. Using built-in database.
   Configuring ambari database...
   ```

7. 启动

   ```
   sudo systemctl start ambari-server.service
   ```

8. 访问

   http://bjrdc18:8080

   admin

   admin

> 详细参考

### 问题

1. at exports.runInThisContext (vm.js:53:16)

   nodejs 的包版本的问题，[使用如下链接的补丁修正](https://github.com/apache/bigtop/pull/628/files)

   ```sh
   vi ambari-admin/src/main/resources/ui/admin-web/package.json
   ```

   ```json
   {
     "name": "adminconsole",
     "version": "0.0.0",
     "dependencies": {},
     "devDependencies": {
       "bower": "1.3.8",
   ```

   ```json
   {
     "name": "adminconsole",
     "version": "0.0.0",
     "dependencies": {},
     "devDependencies": {
       "bower": "1.8.8",
   ```

   

2. ​      [get] Error opening connection java.io.IOException: Server returned HTTP response code: 403 for URL: https://s3.amazonaws.com/dev.hortonworks.com/HDP/centos7/3.x/BUILDS/3.1.4.0-315/tars/hbase/hbase-2.0.2.3.1.4.0-315-bin.tar.gz

   [解决办法](https://stackoverflow.com/questions/64494636/install-ambari-cant-download-hortonworks-hdp-from-amazon-s3)

   将hbase等的包修改为apache官方地址

   vi **apache-ambari-2.7.5-src/ambari-metrics/pom.xml**

   ```xml
      <hbase.tar>https://archive.apache.org/dist/hbase/2.2.6/hbase-2.2.6-bin.tar.gz</hbase.tar>
       <hbase.folder>hbase-2.2.6</hbase.folder>
       <hadoop.tar>https://archive.apache.org/dist/hadoop/core/hadoop-3.0.3/hadoop-3.0.3.tar.gz</hadoop.tar>
       <hadoop.folder>hadoop-3.0.3</hadoop.folder>
       <grafana.folder>grafana-6.4.2</grafana.folder>
       <grafana.tar>https://dl.grafana.com/oss/release/grafana-6.4.2.linux-amd64.tar.gz</grafana.tar>
       <phoenix.tar>https://archive.apache.org/dist/phoenix/phoenix-5.1.1/phoenix-hbase-2.2-5.1.1-bin.tar.gz</phoenix.tar>
       <phoenix.folder>apache-phoenix-5.0.0-alpha-HBase-2.0-bin</phoenix.folder>
   ```

   vi ambari-metrics/ambari-metrics-timelineservice/pom.xml

   ```xml
     <properties>
       <!-- Needed for generating FindBugs warnings using parent pom -->
       <!--<yarn.basedir>${project.parent.parent.basedir}</yarn.basedir>-->
       <protobuf.version>2.5.0</protobuf.version>
       <hadoop.version>3.0.3</hadoop.version>
       <phoenix.version>5.1.1</phoenix.version>
       <hbase.version>2.2.6</hbase.version>
     </properties>
   ```

   ```xml
   <move
                           file="${project.build.directory}/embedded/${phoenix.folder}/phoenix-server-hbase-2.2-${phoenix.version}.jar"
                           tofile="${project.build.directory}/embedded/${hbase.folder}/lib/phoenix-server-hbase-2.2-${phoenix.version}.jar"
                       />
   ```




## 使用

### 新建集群

