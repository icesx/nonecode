Doris
===

## 安装部署

参考地址

https://doris.apache.org/zh-CN/install/install-deploy.html#%E8%BD%AF%E7%A1%AC%E4%BB%B6%E9%9C%80%E6%B1%82

### 编译

#### 准备

1. 工作目录`/cloud`，50G 的空间

2. cpu 四核，否则慢死

3. 内存8G，否则会killed

4. 安装工具

   ```
   sudo yum install -y byacc patch automake libtool make which file ncurses-devel gettext-devel unzip bzip2 zip util-linux wget git python2
   ```

   

#### 编译

1. 安装maven

   省略

2. 下载ldb_toolchain

   从 https://github.com/amosbird/ldb_toolchain_gen/releases 下载`ldb_toolchain_gen.sh`

   解压

   ```
   ./ldb_toolchain_gen.sh /cloud/ldb_toolchain
   ```

   解压完成后会在`/cloud/ldb_toolchain`下创建ldb_toolchain的相关文件

3. 下载doris

   使用git下载doris

   ```
   cd /cloud
   git clone https://github.com/apache/incubator-doris.git
   ```

   

4. 修改环境变量

   ```
   cat << EOF|tee /cloud/incubator-doris/custom_env.sh
   export JAVA_HOME=/home/bjrdc/software/jdk1.8.0_131_x64
   export PATH=$JAVA_HOME/bin:$PATH
   export PATH=/home/bjrdc/software/apache-maven-3.8.5/bin:$PATH
   export PATH=/home/bjrdc/.nvm/versions/node/v12.13.0/bin:$PATH
   export PATH=/cloud/ldb_toolchain/bin:$PATH
   ```

   注：当前doris的java版本为1.8

5. 编译

   ```
   cd /cloud/incubator-doris
   ./build.sh
   ```

   慢等吧，2-4个小时

6. 异常处理

   编译过程中可能出现文件找不到的错误，可以直接注释掉build.sh中的行

   ```
   #    ./build_help_zip.sh
   #    cp -r -p ${DORIS_HOME}/docs/build/help-resource.zip ${DORIS_OUTPUT}/fe/lib/
   ```

7. 编译完成

   编译完成后在`/cloud/incubator-doris/output`下产生编译成功的文件

   .
   ├── apache_hdfs_broker
   │   ├── bin
   │   ├── conf
   │   └── lib
   ├── be
   │   ├── bin
   │   ├── conf
   │   ├── lib
   │   ├── log
   │   ├── storage
   │   └── www
   ├── fe
   │   ├── bin
   │   ├── conf
   │   ├── doris-meta
   │   ├── lib
   │   ├── log
   │   ├── spark-dpp
   │   └── webroot
   └── udf
       ├── include
       └── lib

### 安装

#### 环境准备

准备三台docker容器，角色如下

| 序号 | 节点     | 用途 | 配置              |
| ---- | -------- | ---- | ----------------- |
| 1    | bjrdc105 | fe   | 2c,5g,32g(/cloud) |
| 2    | bjrdc106 | be   | 2c,5g,32g(/cloud) |
| 3    | bjrdc107 | be   | 2c,5g,32g(/cloud) |
| 4    | bjrdc111 | be   | 2c,5g,20g(/cloud) |

#### 部署

```
scp -r fe bjrdc105:/cloud/doris/
scp -r be bjrdc106:/cloud/doris/
scp -r be bjrdc107:/cloud/doris/
```



#### 修改配置文件

每台机器上执行如下命令

```
echo "priority_networks = 172.16.15.0/24" >> /cloud/doris/fe/conf/fe.conf
echo "storage_root_path = /cloud/doris/storage,medium:hdd" >>/cloud/doris/fe/conf/fe.conf
```



#### 修改默认密码

默认用户admin和root均为空密码，为安全期间，可以将密码修改一下。

```sql
SET PASSWORD FOR 'root' = PASSWORD('your_password');
```



#### 参数

网络端口说明

| 实例名称 | 端口名称               | 默认端口 | 通讯方向                     | 说明                                                 |
| -------- | ---------------------- | -------- | ---------------------------- | ---------------------------------------------------- |
| BE       | be_port                | 9060     | FE --> BE                    | BE 上 thrift server 的端口，用于接收来自 FE 的请求   |
| BE       | webserver_port         | 8040     | BE <--> BE                   | BE 上的 http server 的端口                           |
| BE       | heartbeat_service_port | 9050     | FE --> BE                    | BE 上心跳服务端口（thrift），用于接收来自 FE 的心跳  |
| BE       | brpc_port              | 8060     | FE <--> BE, BE <--> BE       | BE 上的 brpc 端口，用于 BE 之间通讯                  |
| FE       | http_port              | 8030     | FE <--> FE，用户 <--> FE     | FE 上的 http server 端口                             |
| FE       | rpc_port               | 9020     | BE --> FE, FE <--> FE        | FE 上的 thrift server 端口，每个fe的配置需要保持一致 |
| FE       | query_port             | 9030     | 用户 <--> FE                 | FE 上的 mysql server 端口                            |
| FE       | edit_log_port          | 9010     | FE <--> FE                   | FE 上的 bdbje 之间通信用的端口                       |
| Broker   | broker_ipc_port        | 8000     | FE --> Broker, BE --> Broker | Broker 上的 thrift server，用于接收请求              |

### 启动

#### fe

```sh
./bin/start_fe.sh --daemon
```

http://bjrdc105:8030/



| 序号 | 帐号  | 密码 |
| ---- | ----- | ---- |
| 1    | admin |      |
| 2    | root  |      |
|      |       |      |



#### be



```sh
./bin/start_be.sh --daemon
```

http://bjrdc106:8060/

### 使用

#### mysql登录

```
mysql -u root -h bjrdc105 -P 9030
```

#### 增加be

```
ALTER SYSTEM ADD BACKEND "bjrdc106:9050";
```

#### 浏览器登录

http://bjrdc105:8030/

## 使用

### 基本命令

#### mysql

```
ALTER SYSTEM ADD BACKEND "bjrdc106:9050";
show proc '/backends';
show proc '/frontends';
show proc '/brokers';
SET PASSWORD FOR 'root' = PASSWORD('your_password');
```

缩容

```
ALTER SYSTEM DROP FOLLOWER[OBSERVER] "fe_host:edit_log_port";
```

## 例子

### streamload

#### txt

1. 创建库

   ```sql
   create database doris_test;
   ```

   

2. 创建表

   ```sql
   CREATE TABLE IF NOT EXISTS load_local_file_test
   (
       id INT,
       age TINYINT,
       name VARCHAR(50)
   )
   unique key(id)
   DISTRIBUTED BY HASH(id) BUCKETS 3;
   ```

   

3. 导入数据

   创建demo.txt文件

   1	18	张三
   2	19	jack
   3	1	been

   注：默认是tab分割

   ```
   curl -u user:passwd -H "label:load_local_file_test" -T /path/to/local/demo.txt http://host:port/api/demo/load_local_file_test/_stream_load
   ```

   - user:passwd 为在 Doris 中创建的用户。初始用户为 admin / root，密码初始状态下为空。
   - host:port 为 BE 的 HTTP 协议端口，默认是 8040，可以在 Doris 集群 WEB UI页面查看。
   - label: 可以在 Header 中指定 Label 唯一标识这个导入任务。

   ```
   curl -u root -H "label:load_local_file_test" -T /home/i/Downloads/demo.txt http://bjrdc106:8040/api/doris_test/load_local_file_test/_stream_load
   ```

   

#### json

1. 创建表

   ```sql
   CREATE TABLE IF NOT EXISTS city_from_json
   (
   id      INT     NOT NULL,
   city    VARCHAR(50) NULL,
   code    INT     NULL
   )
   unique key(id)
   DISTRIBUTED BY HASH(id) BUCKETS 3;
   ```

2. city.json

   ```json
   [
       {"id": 100, "city": "beijing", "code" : 1},
       {"id": 101, "city": "shanghai"},
       {"id": 102, "city": "tianjin", "code" : 3},
       {"id": 103, "city": "chongqing", "code" : 4},
       {"id": 104, "city": ["zhejiang", "guangzhou"], "code" : 5},
       {
           "id": 105,
           "city": {
               "order1": ["guangzhou"]
           }, 
           "code" : 6
       }
   ]
   ```

3. load to table

   ```
   curl --location-trusted -u root -H "format: json" -H "jsonpaths: [\"$.id\",\"$.city\",\"$.code\"]" -H "strip_outer_array: true" -T /home/i/Downloads/city.json http://bjrdc106:8040/api/doris_test/city_from_json/_stream_load
   ```



## 性能测试

