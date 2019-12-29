###基本命令
	docker run -i -t --name hadoop-name0 ubuntu /bin/bash
	docker rm hadoop-name0
	docker start hadoop-name0
	docker attach hadoop-name0
	docker run -i -t --memory=1024m --hostname=hadoop-name0 --name hadoop-name0 ubuntu /home/hadoop/docker/common/autostart.sh
	docker inspect hadoop-name0
	docker rmi 7b7bbf4afbf0
###Docker的特点
	和LXC相比，对container的是隔离的，image是不会发生变化的。而lxc则是改动都会变化到image中。
	可以通过commit一个image，而之后的container使用这个image来创建的方式，实现对image修改的保留。
	sudo docker commit --author="i" --message="ssh docker software hadoop " hadoop-data00 xjgz/ubuntu:v9
	docker stop hadoop-data00
###安装
0. docker安装
	sudo curl -sSL https://get.docker.com/ | sh
	reboot 启动docker
1. 关于自启动
	docker命令的最后一个“/bin/bash”就是这个container启动的时候启动的命令，可以在container中搞一个自启动文件，然后调用它。
	docker run -i -t --hostname=hadoop-data00 --name=hadoop-data00 xjgz/ubuntu:v9 /home/hadoop/docker/common/autostart.sh，通过这个命令调用自启动的时候老是失败，目前尚不知道问题所在，但是使用变通的办法如下：
		docker run -i -t --hostname=hadoop-data00 --name=hadoop-data00 xjgz/ubuntu:v5
		docker exec hadoop-data00 /etc/rc.local	
		【这个rc.local只是在前台执行，故只能去启动ssh，其他的java环境变量需要设置到.profile中】	
		docker attach hadoop-data00
###关于网络
	1. 第一种方式
	docker run -i -t --hostname=hadoop-data00 --name=hadoop-data00 --mac-address=02:42:ac:11:00:00  xjgz/ubuntu:v9
		设置mac地址也不能保证获取的IP地址是ok的。
	虽然无法设置IP地址，但是可以获取ip地址：docker inspect hadoop-data00|grep IPAddress
	2. 第二种方式
	可以通过linux的相关命令为docker的机器增加网络——参考的pi，pwork——一个开源的shell工具，用于为docker的container配置网络，其原理是通过linux的brctl，ip命令为container增加网络设备。
	通过自己编写的shell【模拟的pipwork】可以工作，但是需要注意几点
		A. 如果是要建立一个跨主机的虚拟的内网【如172】，需要启用主机的第二张网卡，虚拟内网通过第二张网卡通讯。
		B. 如果是要建立一个和主机一个网络的集群，那么使用当前网络的网卡即可
		C. 但是一定注意：在使用ip addr del 和ip addr的命令要一次执行，使用\连接，否则会断网，类似如下：
			sudo brctl addbr $brdge;\
			sudo ip link set $brdge up;\
			sudo ip addr add $hostmask dev $brdge;\
			sudo ip addr del $hostmask dev $eth;\
			sudo brctl addif $brdge $eth
		D. 保险的方式是，启用第二张网卡，但是第二章网卡的地址不能和第一张的相同，可能网络不通，有可能为第二张网卡增加网关可能会好【gatway1=.。。。】

###关于几个重要的配置
	hostname	：172.17.42.1
	name		:
	磁盘		:
	内存		:
1. 关于模板
	提交模板：docker commit --message="add rabbitmq" cdc xjgz/cdc:v1
	docker save -o ubuntu_cdc_add_rabbit.tar xjgz/cdc:v1
	docker rmi xjgz/cdc:v1
	docker load -i images/ubuntu_cdc_v1_add_rabbit.tar

2. 修改docker的image的路径的方法
		I. Ubuntu/Debian: edit your /etc/default/docker file with the -g option: DOCKER_OPTS="-dns 8.8.8.8 -dns 8.8.4.4 -g /mnt"
		II. rm -rf /lib/systemd/system/docker.service
		III. systemctl daemon-reload
		IV. service docker restart
3. 关于挂载外部硬盘
	docker run -i -t --hostname=hadoop-data00 --name=hadoop-data00 --link=hadoop-name00:hadoop-name00 --volume=/home/docker/software:/home/docker/software:rw --volume=/home/docker/volume/data00:/home/docker/volume:rw  xjgz/ubuntu:v9