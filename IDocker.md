Docker
========
### 基本命令
```
docker run -i -t --name hadoop-name0 ubuntu /bin/bash
docker run -p 8080:8080 bjrdc-dev/spring-cloud-eureka:0.0.1-SNAPSHOT
docker run -p 127.0.0.1:8080:8080 bjrdc-dev/spring-cloud-eureka:0.0.1-SNAPSHOT
#坑点：-p参数不能放后面，必须放到run后面

docker rm hadoop-name0
docker start hadoop-name0
docker attach hadoop-name0
docker run -i -t --memory=1024m --hostname=hadoop-name0 --name hadoop-name0 ubuntu /home/hadoop/docker/common/autostart.sh
docker inspect hadoop-name0
docker rmi 7b7bbf4afbf0
docker rm b5b7ead8f2fc
#删除ps
```


```
docker login bjrdc206:443
docker tag hello-world bjrdc206:443/bjrdc-dev/hello-world:v1.0.0
docker push bjrdc206:443/bjrdc-dev/hello-world:v1.0.0
```



### Docker的特点

	和LXC相比，对container的是隔离的，image是不会发生变化的。而lxc则是改动都会变化到image中。
	可以通过commit一个image，而之后的container使用这个image来创建的方式，实现对image修改的保留。
	sudo docker commit --author="i" --message="ssh docker software hadoop " hadoop-data00 xjgz/ubuntu:v9
	docker stop hadoop-data00
### 安装
0. docker安装
```
sudo curl -sSL https://get.docker.com/ | sh
reboot 启动docker
```
1. 关于自启动
docker命令的最后一个“/bin/bash”就是这个container启动的时候启动的命令，可以在container中搞一个自启动文件，然后调用它。
```
docker run hello-world
```



```
docker run -i -t --hostname=hadoop-data00 --name=hadoop-data00 xjgz/ubuntu:v9 /home/hadoop/docker/common/autostart.sh，通过这个命令调用自启动的时候老是失败，目前尚不知道问题所在，但是使用变通的办法如下：
```
```
docker run -i -t --hostname=hadoop-data00 --name=hadoop-data00 xjgz/ubuntu:v5
docker exec hadoop-data00 /etc/rc.local	
【这个rc.local只是在前台执行，故只能去启动ssh，其他的java环境变量需要设置到.profile中】	
docker attach hadoop-data00
```
### 关于网络
>1. 第一种方式
>docker run -i -t --hostname=hadoop-data00 --name=hadoop-data00 --mac-address=02:42:ac:11:00:00  xjgz/ubuntu:v9
>设置mac地址也不能保证获取的IP地址是ok的。
>虽然无法设置IP地址，但是可以获取ip地址：docker inspect hadoop-data00|grep IPAddress



>2. 第二种方式
>  可以通过linux的相关命令为docker的机器增加网络——参考的pi，pwork——一个开源的shell工具，用于为docker的container配置网络，其原理是通过linux的brctl，ip命令为container增加网络设备。
>  通过自己编写的shell【模拟的pipwork】可以工作，但是需要注意几点
>
>  1. 如果是要建立一个跨主机的虚拟的内网【如172】，需要启用主机的第二张网卡，虚拟内网通过第二张网卡通讯。
>  2. 如果是要建立一个和主机一个网络的集群，那么使用当前网络的网卡即可
>  3. 但是一定注意：在使用ip addr del 和ip addr的命令要一次执行，使用\连接，否则会断网，类似如下：
>
>  ```
>  sudo brctl addbr $brdge;\
>  sudo ip link set $brdge up;\
>  sudo ip addr add $hostmask dev $brdge;\
>  sudo ip addr del $hostmask dev $eth;\
>  sudo brctl addif $brdge $eth
>  ```
>
>  4. 保险的方式是，启用第二张网卡，但是第二章网卡的地址不能和第一张的相同，可能网络不通，有可能为第二张网卡增加网关可能会好【gatway1=.。。。】
### 关于几个重要的配置
hostname	：172.17.42.1
name		:
磁盘		:
内存		:
### 关于模板
1. 提交模板：
```
docker commit --message="add rabbitmq" cdc xjgz/cdc:v1
docker save -o ubuntu_cdc_add_rabbit.tar xjgz/cdc:v1
docker rmi xjgz/cdc:v1
docker load -i images/ubuntu_cdc_v1_add_rabbit.tar
```
### 修改docker的image的路径的方法

### 17.09-ce之前

Ubuntu/Debian:
edit your /etc/default/docker file with the -g option:
```
DOCKER_OPTS="-dns 8.8.8.8 -dns 8.8.4.4 -g /docker"
rm -rf /lib/systemd/system/docker.service
systemctl daemon-reload
service docker restart
```
#### 17.09-ce之后

```
cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://registry.docker-cn.com"],
  "graph": "/docker",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```



### 关于挂载外部硬盘

```
docker run -i -t --hostname=hadoop-data00 --name=hadoop-data00 --link=hadoop-name00:hadoop-name00 --volume=/home/docker/software:/home/docker/software:rw --volume=/home/docker/volume/data00:/home/docker/volume:rw  xjgz/ubuntu:v9
```

### 乱码

在开发项目的时候用到了docker，当系统部署到docker上，发现有乱码的现象，所有的汉字都变成了？，经过对比测试发现

> 第一、本机的linux操作系统上是OK的
> 第二、本机的linux的lcale是LANG=zh_CN.UTF-8，docker上的ubuntu也是LANG=zh_CN.UTF-8
> 第三、讲docker上的LANG修改为LANG=en_US.UTF-8后依旧
> 第四、此时可以肯定的是环境的问题，那是什么环境的问题呢？
> 第五、将java的所有的properties打印出来后发现，字符的默认编码是US-ASCII，这个非常的异常
> 第六、经过多种转发后仍然不行，但是getbutes(UTF-8)后的byte[]直接写到文件后，是中文的。
> 第七、试着修改Java的默认编码使用 java -Dencode.file=UTF-8,发现问题解决
> 第八、最终解决办法，在catalina.sh中增加JAVA_OPTS="-Dfile.encoding=UTF-8"

### 非sudo执行

> docker默认需要sudo执行，可通过如下命令使其非sudo执行
>
> ```
> sudo usermod -a -G docker jenkins
> newgrp docker
> docker run hello-world
> ```
>

### Error response from daemon: Get https://bjrdc206.reg/v2/: x509: certificate signed by unknown authority

> add code to /etc/docker/daemon.json
>
> `"insecure-registries":["bjrdc206.reg"]`
>
> ```
> sudo service docker restart
> ```

## Dockerfile

```
cat > Dockerfile <<EOF
FROM java:8-jdk
ARG JAR_FILE
ADD target/${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
EOF
docker build -t xx:v1 .
```





## docker with hadoop

```
#!/bin/bash
dockers_hadoop="hadoop-name01 hadoop-data00 hadoop-data01 hadoop-data02 hadoop-data03 hadoop-data04 hadoop-data05 hadoop-data06"
dockers_hbase="hbase00 hbase01"
dockers_zookeeper="zookeeper00 zookeeper01"
docker_image="xjgz/ubuntu:v9"

###### common start
function run_docker(){
	docker=$1
	_docker=$(sudo docker ps|grep $docker)
	if [ $(echo $_docker|wc -L) -eq 0 ] ; then
		echo ":::docker "$docker" is not started, and check is it created"
		docker_created=$(sudo docker ps -a|grep $docker)
		if [ $(echo $docker_created|wc -L) -eq 0 ];then
			echo ":::docker "$docker" is not created so to run it"
			sudo docker run -i -t -d --hostname=$docker --name=$docker --volume=/home/docker/software:/home/docker/software:rw --volume=/home/docker/volume/$docker:/home/docker/volume:rw  $docker_image
			sleep 1s
		else
			echo ":::docker "$docker" is created so to start it!"
			sudo docker start $docker
			sleep 1s
		fi
		change_host $docker
		start_service $docker
	else
		echo ":::docker "$docker" is started !"
	fi	
}
function change_host(){
	docker=$1;
	ip=$(sudo docker inspect $docker |grep IPAddress|awk -F "\"" '{print $4}')
	echo ":::docker "$docker" ip is "$ip
	change_host_ip $docker $ip
}
function change_host_ip(){
	docker=$1
	ip=$2
	_hosts=$(grep -e "[0-9]\{1,3\}.[0-9]\{1,3\}.[0-9]\{1,3\}.[0-9]\{1,3\}[     | ]"$docker /etc/hosts)
	if [ $(echo $_hosts|wc -L) -eq 0 ];then
			echo ":::echo $ip	$docker to/etc/hosts"
			sudo sh -c "echo $ip	$docker>>/etc/hosts"
	else
		old_ip=$(echo $_hosts|awk '{print $1}')
		echo ":::host "$docker" old ip is "$old_ip" change to "$ip
		sudo sed -i "s/"$old_ip"/"$ip"/g" /etc/hosts
	fi
}
function start_service(){
	docker=$1
	echo ":::to start "$docker "'s service by /etc/local"
	sudo docker exec $docker /etc/rc.local
}
function stop_docker(){
	docker=$1
	sudo docker stop $docker
}

function delete_docker(){
	docker=$1
	echo ":::to delete docker"
	stop_docker $docker
	sudo docker rm $docker
}
function start_dockers_by_array(){
	docker_array=$1
	for docker in $docker_array
		do 
			echo ":::start "$docker
			run_docker $docker
		done
}
function stop_dockers_by_array(){
	docker_array=$1
	for docker in $docker_array
		do 
			echo ":::stop "$docker
			stop_docker $docker
		done
}
function delete_dockers_by_array(){
	docker_array=$1
	for docker in $docker_array
		do 
			echo ":::delete "$docker
			delete_docker $docker
		done
}
function run_dockers_shell(){
	docker_array=$1
	shell=$2
	for docker in $docker_array
		do 
			echo ":::run shell "$shell" on" $docker
			sudo docker exec $docker $shell
		done
}
function run_ssh_shell(){
	docker_array=$1
	shell=$2
	for docker in $docker_array
		do 
			echo ":::run shell by ssh "$shell" on" $docker
			ssh "docker@"$docker "'$shell'"
		done
}
function rsync_hosts(){
	docker_array=$1
	for docker in $docker_array
		do 
			echo ":::scp /etc/hosts to "$docker
			scp /etc/hosts "docker@"$docker:/home/docker
		done
	run_dockers_shell "$docker_array" /home/docker/shell/rc.sh
}
######## common end
######## for hadoop start
function start_hadoop_docker(){
	start_dockers_by_array "$dockers_hadoop"
}
function delete_hadoop_docker(){
	delete_dockers_by_array "$dockers_hadoop"
}
function stop_hadoop_docker(){
	stop_hadoop_service
	stop_dockers_by_array "$dockers_hadoop"
}
function start_hadoop_service(){
	echo ":::to start hadoop"
	/home/docker/software/hadoop-2.6.0/sbin/start-dfs.sh
	/home/docker/software/hadoop-2.6.0/sbin/start-yarn.sh
}
function stop_hadoop_service(){
	echo ":::to stop hadoop"
	/home/docker/software/hadoop-2.6.0/sbin/stop-dfs.sh
	/home/docker/software/hadoop-2.6.0/sbin/stop-yarn.sh
}
function start_hadoop(){
	start_hadoop_docker
	start_hadoop_service
}
function stop_hadoop(){
	stop_hadoop_service
	stop_hadoop_docker
}
######### for hadoop end
######### for hbase start
function start_hbase_docker(){
	start_dockers_by_array "$dockers_hbase"
	rsync_hosts "$dockers_hbase"
}
function stop_hbase_docker(){
	stop_dockers_by_array "$dockers_hbase"
}
function delete_hbase_docker(){
	delete_dockers_by_array "$dockers_hbase"
}
function start_hbase_service(){
	echo ":::to start hbase"
	/home/docker/software/hbase-0.98.12/bin/start-hbase.sh
}
function stop_hbase_service(){
	echo ":::to stop hbase"
	/home/docker/software/hbase-0.98.12/bin/stop-hbase.sh
}
function start_hbase(){
	start_zookeeper
	start_hbase_docker
	start_hbase_service
}
function stop_hbase(){
	stop_hbase_service
	stop_hbase_docker
}
######### for hbase end
######### for zookeeper start
function start_zookeeper_service(){
	echo ":::to start zookeeper"
	run_ssh_shell "$dockers_zookeeper" /home/docker/shell/zookeeper_start.sh
}
function stop_zookeeper_service(){
	echo ":::to stop zookeeper"
	run_ssh_shell "$dockers_zookeeper" /home/docker/shell/zookeeper_stop.sh
}
function start_zookeeper_docker(){
	start_dockers_by_array "$dockers_zookeeper"
	rsync_hosts "$dockers_zookeeper"
}
function stop_zookeeper_docker(){
	stop_dockers_by_array "$dockers_zookeeper"
}
function delete_zookeeper_docker(){
	delete_dockers_by_array "$dockers_zookeeper"
}
function start_zookeeper(){
	start_zookeeper_docker
	start_zookeeper_service
}
function stop_zookeeper(){
	stop_zookeeper_service
	stop_zookeeper_docker
}
######### for zookeeper end
######### for all docker
function start_all_docker(){
	start_hadoop_docker
	start_hbase_docker
	start_zookeeper_docker
}
function stop_all_docker(){
	stop_hadoop_docker
	stop_hbase_docker
	stop_zookeeper_docker
}
function delete_all_docker(){
	delete_hadoop_docker
	delete_hbase_docker
	delete_zookeeper_docker
}
#########
######### main
function show_usage(){
	echo "Usage: auto_docker.sh COMMAND ARG"
	echo "COMMAND:"
	echo "	all		will stop or start all hadoop hbase zookeeper docker"
	echo "	hadoop		will stop or start hadoop service"
	echo "	hbase		will stop or start hadoop service,zookeeper service then start hbase service"
	echo "	zookeeper	will stop or start zookeeper service only"
	echo "	docker_ha	will stop or start docker for hadoop"
	echo "	docker_hb	will stop or start docker for hbase"
	echo "	docker_zo	will stop or start docker for zookeeper"
	echo "	docker_al	will stop or start docker for hadoop hbase zookeeper"	
	echo "ARG:"
	echo "	start		start service or docker"
	echo "	stop 		stop service or docker"
	echo "	delete		delete docker command must by docker_xx"
}
function main(){
	command=$1
	arg=$2
	sudo echo "command is "$command" arg is "$arg
	if [ $(echo $command|wc -L) -eq 0 ];then
		show_usage
	elif [ $(echo $arg|wc -L) -eq 0 ];then
		show_usage	
	else
		if [ $command = "all" ];then
			if [ $arg = "start" ];then
				start_dockers
				start_hadoop
				start_zookeeper
				start_hbase
			elif [ $arg = "stop" ];then
				stop_hbase
				stop_zookeeper
				stop_hadoop
				stop_dockers
			else
				show_usage
			fi
		elif [ $command = "hadoop" ];then
			if [ $arg = "start" ];then
				start_hadoop
			elif [ $arg = "stop" ];then
				stop_hadoop
			else
				show_usage
			fi
		elif [ $command = "hbase" ];then
			if [ $arg = "start" ];then
				start_hbase
			elif [ $arg = "stop" ];then
				stop_hbase
			else
				show_usage
			fi
		elif [ $command = "zookeeper" ];then
			if [ $arg = "start" ];then
				start_zookeeper
			elif [ $arg = "stop" ];then
				stop_zookeeper
			else
				show_usage
			fi
		elif [ $command = "docker_ha" ];then
			if [ $arg = "start" ];then
				start_hadoop_docker
			elif [ $arg = "stop" ];then
				stop_hadoop_docker
			elif [ $arg = "delete" ];then
				delete_hadoop_docker
			else
				show_usage
			fi
		elif [ $command = "docker_hb" ];then
			if [ $arg = "start" ];then
				start_hbase_docker
			elif [ $arg = "stop" ];then
				stop_hbase_docker
			elif [ $arg = "delete" ];then
				delete_hbase_docker
			else
				show_usage
			fi
		elif [ $command = "docker_zo" ];then
			if [ $arg = "start" ];then
				start_zookeeper_docker
			elif [ $arg = "stop" ];then
				stop_zookeeper_docker
			elif [ $arg = "delete" ];then
				delete_zookeeper_docker
			else
				show_usage
			fi
		elif [ $command = "docker_al" ];then
			if [ $arg = "start" ];then
				start_all_docker
			elif [ $arg = "stop" ];then
				stop_all_docker
			elif [ $arg = "delete" ];then
				delete_all_docker
			else
				show_usage
			fi
		else
			show_usage
		fi
	fi
}
main $1 $2
```

