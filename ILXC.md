###debootstrap 
0. virtualbox的配置
	A. 首先安装ubuntu14.10版本，其他的版本应该也可以，本文档基于ubuntu14.10测试通过
	B. 安装的虚拟机配置两个网卡，一个使用host-only【用于远程登录】，一个采用“网络地址转换NAT”【用于连接外网，经过测试后发现，“NAT”模式不行，无法从虚拟机中ping出外网】
	C. 虚拟机做一些精简【不必须】
1. 安装相关软件
	A. sudo apt-get install lxc bridge-utils debootstrap libcap-dev
		#安装需要的程序文件 
2. 下载container文件
	A. sudo debootstrap --arch amd64 utopic rootfs.ubuntu.utopic http://ubuntu.cn99.com/ubuntu
		#将从http://archive.ubuntu.com/ubuntu/dists/utopic 下载文件，下载到当前目录的rootfs.ubuntu.utopic文件夹下
		#--arch amd64 指令集平台
		#sudo debootstrap –arch [平台] [发行版本代号] [目录] [url]
		#--variant=minbase,最小安装，minbase|buildd|fakechroot|scratchbox
	B. 网上有说明可以直接使用lxc-create来创建container，但是本文未使用，待日后测试。
3. container基本相关配置
	A. chroot /lxc/rootfs.ubuntu /bin/bash
		#进入container
	B. apt-get install openssh-server vim rsyslog sudo
	C. echo "ilxc" > /etc/hostname
	D. echo "127.0.0.1 localhost" > /etc/hosts
	E. exit
		#经过测试如上A-E的配置可以不做，具体有什么问题尚未验证。
4. container设备等配置
	A. cd rootfs.ubuntu.utopic/
	B. sh home/ilxc/iLxc-config.sh
		#如上A-B的设置业可以不用做，具体有什么问题尚未验证。
5. 设置网络【重点】
	A. ifconfig查看虚拟机网络
		I. eth1：10.0.4.15
		II. lxcbr0：10.0.3.X【此网桥为安装bridge-utils后自动产生的一个网桥，不用删除它，可以修改ip使用】
		III. eth0：192.168.56.102【host-only分配的ip地址】
		IV. route命令查看虚拟机的默认路由为10.0.4.2
	B. 在虚拟机上运行如下命令，修改网桥配置
		I. brctl addif lxcbr0 eth1
			#将eth1加入网桥
		II. ifconfig eth1 0.0.0.0
			#eth1不需要ip地址了
		III. ifconfig lxcbr0 10.0.4.15
			#讲网桥的地址设置为之前eth1的地址【理论上来说，其他的地址10.0.4.×地址也可以】
		IV. route add -net default gw 10.0.4.2
			#修改网桥配置后，默认路由地址发生变化，要手动设置回来。
			#为网桥所在的地址段增加默认路由,让所有的网络包默认通过网桥
		V. ping baidu.com,正常情况下，网络可以通，如果不同试试直接ping外网地址。
6. 运行container
	A. lxc-execute -n ubuntu -f config.ubuntu -- /bin/bash
	B. 通过ifconfig查看container网络
		10.0.4.101，为config.ubuntu中设置的ip地址
	C. 为container增加默认路由
		route add  -net default gw 10.0.4.15
			#10.0.4.15为虚拟机的网桥的地址
	D. 设置DNS
		I. /etc/resolv.conf 中增加
			nameserver 10.0.4.15【理论上说，可以通过10.0.4.15解析域名，但是实地测试不生效】
			nameserver 8.8.8.8【增加google的DNS】
		II. ping baidu.com,OK
	E. 由于没有进行系统设备的设置，可能无法进行远程登录等动作，需要后续安装相关的资料再进行设置即可
7. 进行享受吧,但是发现有时可以，有时不行，不知道是不是virtualbox的问题
8. 试验了两天发现这种方式部署的container不是很稳定，偶发性的网络问题。
9. 下一步试试lxc-create


###lcx-create
0. 之前经过debootstrap来创建container发现是可以的，但是发现总是不够稳定。下面将使用lxc-create试试
1. 通过lxc-create命令创建的container就很简单，详细如下：
	A. sudo lxc-create -t ubuntu -n ubuntu
	B. sudo lxc-start -n ubuntu -d
	C. sudo lxc-console -n ubuntu
2. 经过测试发现lxc-create创建的container也可以移动到指定的目录，使用如下方法：
	A. mv  /var/lib/lxc/ubuntu/ /DOING/LXC/
	B. lxc-start -f config -n ubuntu -d
	C. 以此推理，采用 sudo lxc-create -t ubuntu -n master --dir=/home/i/lxc/master，也可以。
	D. 相关的配置文件在$(pwd)/lxc-ok/
3. 基本的概念
	A. 一个container可以创建多个实例，通过lxc-start 来启动实例
	B. 如果container相同实例启动之后文件将相同，为了达到不同的实例不同的文件，则需要对不同的实例mount不同的目录
4. 配置IP地址
	A. 默认启动的实例的IP地址为DHCP的
	B. container实例配置
		lxc.utsname=hadoop01【目前还没有发现有什么作用】
		lxc.include=/DOING/LXC/ubuntu/config【公共的一些配置，网络参数】
		lxc.network.hwaddr=00:16:3e:00:00:01【mac地址】
		lxc.network.ipv4=10.0.3.101/24【静态IP地址，需要和lxcbr0在一个网段】
		lxc.include=/DOING/LXC/ubuntu/fstab【需要mount的公共文件系统】
		lxc.include=/DOING/LXC/hadoop01/fstab【需要mount的文件系统】
	C. ubuntu/config【此congfig文件在lxc-create的时候自动创建，后需要修改网络等参数】
		lxc.include=/usr/share/lxc/config/ubuntu.common.conf【引用公共的ubuntu的配置文件】
		lxc.rootfs =/DOING/LXC/ubuntu/rootfs.ubuntu【container目录】
		lxc.network.type = veth
		lxc.network.flags = up
		lxc.network.link = lxcbr0【网桥名称】
		lxc.network.name = eth0
		lxc.network.mtu = 1500
		lxc.network.ipv4.gateway = auto【自动获得网管】
	D. hadoop01/fstab
		lxc.mount.entry=/TOOLS/hadoop 			home/ubuntu/hadoop 			none 	ro,bind 0 0
		【源文件目录】					【需要挂载到的实例的目录，一定要存在】				
	E. ubuntu/fstab
		A. lxc.mount.entry=/TOOLS/jdk1.7.0_71_linux_x64 home/ubuntu/jdk1.7.0_71_linux_x64   none bind,ro 0 0
		
5. 启动实例
	A. 启动
		cd /DOING/LXC/hadoop01/
		sudo lxc-start -n hdoop01 -f config -d
	B. 查看配置
		A. 使用PAC或者 lxc-console登录到实例【账户密码为：ubuntu:ubuntu】
		B. df查看分区，可能只能看到一个hadoop的挂载点，但是cd jdk1.7.0_71_linux_x64，却能看到内容——估计是一个bug
		C. mount的代码默认使用ubuntu账户，需要的话，可以chmod -R i:i /TOOLS/hadoop修改hadoop的目录权限给i账户
	C. 这样基本就配置好了，下一步就是将hadoop跑起来了。
	 
6. 修改root密码 $ sudo lxc-attach -n hadoop01
7. 关于lxc.utsname=无法生效的问题，经过无意间的测试发现，将模板ubuntu的hostName修改为非ubuntu则外部的这个utsname就可以生效了，奇哉怪哉



