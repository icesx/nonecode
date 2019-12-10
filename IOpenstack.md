openstack
============

### 前提：centos
1. 在所有的节点上
	sudo systemctl disable NetworkManager $ sudo systemctl enable network $ sudo systemctl stop NetworkManager.service $ sudo systemctl start network.service
2. 在主节点上执行如下：
	yum search centos-release-openstack
	[root@hadoop100 ~]# yum search centos-release-openstack
	Loaded plugins: fastestmirror
	Loading mirror speeds from cached hostfile
	 * base: mirrors.sina.cn
	 * extras: mirrors.yun-idc.com
	 * updates: mirrors.sina.cn
	================================================== N/S matched: centos-release-openstack ===================================================
	centos-release-openstack.noarch : OpenStack from the CentOS Cloud SIG repo configs
	centos-release-openstack-kilo.noarch : OpenStack from the CentOS Cloud SIG repo configs
	centos-release-openstack-liberty.noarch : OpenStack from the CentOS Cloud SIG repo configs
	centos-release-openstack-mitaka.noarch : OpenStack from the CentOS Cloud SIG repo configs
	【centos-release-openstack-mitaka为最新版本】
	yum install -y centos-release-openstack-mitaka
	yum update -y
	yum install -y openstack-packstack
3. 无密码登陆
	。。。。。。
4. controller
	packstack --install-hosts=hadoop100,hadoop100,hadoop50,hadoop1,hadoop150
	packstack --install-hosts=hadoop100,hadoop100,hadoop50,hadoop1,hadoop150 --cinder-volumes-create=n --cinder-host=hadoop100,hadoop50,hadoop1,hadoop150
5. 问题处理
	A、mysql与mariadb的冲突——centos7之后使用的默认数据库是mariadb【虽然命令仍然是mysql】，有可能资源里一斤有mysql了，此时需要
		#rm  -rf /etc/yum.repos.d/mysql-*
		#yum clean dbcache
	B、root无法登陆的问题
		修改mariadb的默认密码为空密码
6. 关于LVM
	默认情况下使用packstack安装openstack的时候，它会生成一个镜像文件在/var/libcinder/cinder-volumes，默认大小为20GB
	[在安装的时候需要让packstack不要生成这个卷，而使用自己构建的卷- --cinder-volumes-create=n】
	这个cinder-volumes会映射到/dev/loop[0-9]下，如/dev/loop2
		[root@hadoop100 cinder]# losetup 
		NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE
		/dev/loop0         0      0         1  0 /var/lib/docker/devicemapper/devicemapper/data
		/dev/loop1         0      0         1  0 /var/lib/docker/devicemapper/devicemapper/metadata
		/dev/loop2         0      0         0  0 /var/lib/cinder/cinder-volumes
		/dev/loop3         0      0         1  0 /srv/loopback-device/swiftloopback
	

			  --- Volume group ---
		  VG Name               cinder-volumes
		  System ID             
		  Format                lvm2
		.
		 .  
		  .
		  --- Physical volumes ---
		  PV Name               /dev/loop2     
		  PV UUID               uB8KGN-9zH0-kfeq-swva-KAOU-fAkU-2Dvpih
		  PV Status             allocatable
		  Total PE / Free PE    5273 / 5273
		之后会再将这个/dev/loop2做成一个lvm：cinder-volumes
	按照这套原理，需要增加cinder-volumes的大小需要：
	A、首先使用lvm，调整根分区“/”大小
	B、使用如下命令调整相关的文件大小和虚拟卷大小
		#truncate -s +900G cinder-volumes
		#losetup -fv cinder-volumes
		#pvresize /dev/loop2
		#losetup -d /dev/loop2 可以删除loop设备，但是一定记住不要使用rm 删除/dev/loop2

7. 问题
	openstack只支持lvm以iscsc的方式，哎

	1. 本地存储

　　		对于本地存储，cinder-volume可以使用lvm驱动，该驱动当前的实现需要在主机上事先用lvm命令创建一个cinder-volumes的vg, 当该主机接受到创建卷请求的时候，cinder-volume在该vg上创建一个LV, 并且用openiscsi将这个卷当作一个iscsi tgt给export.
		当然还可以将若干主机的本地存储用sheepdog虚拟成一个共享存储，然后使用sheepdog驱动。
7、卸载openstack

yum -y remove openstack-* openvswitch memcached httpd python-neutron rabbitmq-server
