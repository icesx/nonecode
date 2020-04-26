rabbitmq
=========
### 安装
+ 在ubuntu下安装rabbit，使用apt-get就可以
```
#sudo apt-get install rabbitmq-server
或者安装最新版本：
#echo 'deb http://www.rabbitmq.com/debian/ testing main' |sudo tee /etc/apt/sources.list.d/rabbitmq.list
#wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc |
#sudo apt-key add -
#sudo apt-get update
#sudo apt-get install rabbitmq-server
```
+ 如果需要编译的话比较麻烦，需要先安装zip、xsltproc、xmlto，其中xmlto很大，所以放弃了。编译按照如下命令：
```
#make install TARGET_DIR=/opt/mq/rabbitmq SBIN_DIR=/opt/mq/rabbitmq/sbin MAN_DIR=/opt/mq/rabbitmq/man 
```
### 启动	
+ 安装好后，使用如下命令启动
```
rabbitmq-server -detached	
rabbitmq-plugins enable rabbitmq_management
#rabbitmq-server stop
#rabbitmq-server -detached
```
+ + 第一次安装，可能要重启虚拟机
### 集群启动
如果需要集群启动，需要在每个主机上运行如下命令
```
#rabbitmq-server -detached
#rabbitmqctl stop_app
#rabbitmqctl join_cluster rabbit@hadoop9
#rabbitmqctl start_app
#rabbitmqctl cluster_status
```
### 异常处理
+ 集群无法启动
有类似如下错误：	Error: unable to connect to  nodedown TCP connection succeeded but Erlang distribution failed
原因：			cookes的同步问题，具体原理尚不知道。
解决办法：		
    1. 将集群中的一个服务器上的/var/lib/rabbit/.erlang.cookies文件scp到其他的服务器上【注意文件的属主一定不能更改，如果在scp的时候改了的话，需要改成rabbitmq的属主】
    2. 重启rabbit-server
+ guest无法登录web界面
````
tail -f /var/log/rabbitmq/rabbit\@hadoop-cq83.log 
user 'guest' - User can only log in via localhost

So I created the **rabbitmq.config** inside the directory /etc/rabbitmq with this:
[{rabbit, [{loopback_users, []}]}].

{后面的.不能少}
service rabbitmq-server restart
centos: rabbitmq-server stop
centos:	rabbitmq-server -detached
```
+ guest 无法登录web界面3.3.0版本之后
This is a new features since the version 3.3.0. 
```
sudo rabbitmqctl add_user test test
sudo rabbitmqctl set_user_tags test administrator
sudo rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
```
+ rabbitmqctl reset
no_running_cluster_nodes,"You cannot leave a cluster if no online nodes are present."
rabbitmqctl force_reset 

### log
	/var/log/rabbitmq/rabbit@hadoop9.log

### install on centos
	A、Centos如果没有接互联网的时候无法用yum安装rabbit，需要使用rpm包
	B、首先下载rabbitMQ的安装包，如
### 关于持久化
	持久化的原理是在在创建queue的时候设置durable=true,然后在receve中，设置autoack=false，在处理完消息后，手动调用channel.basicAck().
### 一个消息多个消费


### RPM安装
```
wget https://packages.erlang-solutions.com/erlang/esl-erlang/FLAVOUR_1_general/esl-erlang_18.3-1~centos~7_amd64.rpm
wget https://github.com/jasonmcintosh/esl-erlang-compat/releases/download/1.1.1/esl-erlang-compat-18.1-1.noarch.rpm
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.5.7-1.noarch.rpm
wget https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
rpm --import rabbitmq-signing-key-public.asc
```
如果出现少了某个so文件，在google去直接搜索这个so文件，然后去下载这个rpm
```
wget ftp://ftp.pbone.net/mirror/ftp.centos.org/7.2.1511/cloud/x86_64/openstack-kilo/common/wxGTK-gl-2.8.12-8.el7.x86_64.rpm
wget ftp://ftp.pbone.net/mirror/ftp.centos.org/7.2.1511/cloud/x86_64/openstack-mitaka/common/wxGTK-2.8.12-8.el7.x86_64.rpm
wget ftp://ftp.pbone.net/mirror/ftp.centos.org/7.2.1511/cloud/x86_64/openstack-liberty/common/wxBase-2.8.12-8.el7.x86_64.rpm
yum install lksctp-tools tk graphive unixODBC libGL libGLU rpm-build SDL
yum install wxBase-2.8.12-8.el7.x86_64.rpm
yum install wxGTK-gl-2.8.12-8.el7.x86_64.rpm
yum install wxGTK-2.8.12-8.el7.x86_64.rpm
yum install esl-erlang_18.3-1-centos-7_amd64.rpm
yum install esl-erlang-compat-18.1-1.noarch.rpm
yum install rabbitmq-server-3.6.1-1.noarch.rpm

/var/log/rabbitmq/rabbit
```
### 修改存储目录

