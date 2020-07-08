LXC
======
## chroot

> chroot 即 change root directory (更改 root 目录)。在 linux 系统中，系统默认的目录结构都是以 /，即以根 (root) 开始的。而在使用 chroot 之后，系统的目录结构将以指定的位置作为 / 位置。

安全性

> 在经过 chroot 之后，在新根下将访问不到旧系统的根目录结构和文件，这样就增强了系统的安全性。一般会在用户登录前应用 chroot，把用户的访问能力控制在一定的范围之内。

基本命令

>```
>sudo chroot /path/to/new/root /bin/bash
>```

## debootstrap 

>debootstrap是debian/ubuntu下的一个工具，用来构建一套基本的系统(根文件系统)。生成的目录符合Linux文件系统标准(FHS)，即包含了/boot、/etc、/bin、/usr等等目录s

1. virtualbox的配置
    A. 首先安装ubuntu14.10版本，其他的版本应该也可以，本文档基于ubuntu14.10测试通过
    B. 安装的虚拟机配置两个网卡，一个使用host-only【用于远程登录】，一个采用“网络地址转换NAT”【用于连接外网，经过测试后发现，“NAT”模式不行，无法从虚拟机中ping出外网】
    C. 虚拟机做一些精简【不必须】
2. 安装相关软件

  > ​	安装需要的程序文件 

  ```
  sudo apt-get install lxc bridge-utils debootstrap libcap-dev
  ```

  

2. 下载container文件

  > #将从http://archive.ubuntu.com/ubuntu/dists/utopic 下载文件，下载到当前目录的rootfs.ubuntu.utopic文件夹下

  ```
sudo debootstrap --arch amd64 utopic rootfs.ubuntu.utopic http://ubuntu.cn99.com/ubuntu
  ```

  > ​	#--arch amd64 指令集平台
  > ​	#sudo debootstrap –arch [平台] [发行版本代号] [目录] [url]
  > ​	#--variant=minbase,最小安装，minbase|buildd|fakechroot|scratchbox

  > 网上有说明可以直接使用lxc-create来创建container，但是本文未使用，待日后测试。

3. container基本相关配置

  >进入container
  >
  >```
  >chroot /lxc/rootfs.ubuntu /bin/bash
  >```
  >
  >安装container内的软件
  >
  >```
  >apt-get install openssh-server vim rsyslog sudo
  >echo "ilxc" > /etc/hostname
  >echo "127.0.0.1 localhost" > /etc/hosts
  >exit
  >```
  >
  >

4. container设备等配置
    A. cd rootfs.ubuntu.utopic/
    B. sh home/ilxc/iLxc-config.sh
    	#如上A-B的设置业可以不用做，具体有什么问题尚未验证。

### 设置网络【重点】

> ifconfig查看虚拟机网络
>
> eth1：10.0.4.15
> lxcbr0：10.0.3.X【此网桥为安装bridge-utils后自动产生的一个网桥，不用删除它，可以修改ip使用】
> eth0：192.168.56.102【host-only分配的ip地址】
> route命令查看虚拟机的默认路由为10.0.4.2



> 在虚拟机上运行如下命令，修改网桥配置
>
> ```
> brctl addif lxcbr0 eth1
> #将eth1加入网桥
> ifconfig eth1 0.0.0.0
> #eth1不需要ip地址了		
> ifconfig lxcbr0 10.0.4.15
> #讲网桥的地址设置为之前eth1的地址【理论上来说，其他的地址10.0.4.×地址也可以】
> route add -net default gw 10.0.4.2
> #修改网桥配置后，默认路由地址发生变化，要手动设置回来。
> #为网桥所在的地址段增加默认路由,让所有的网络包默认通过网桥
> ping baidu.com,正常情况下，网络可以通，如果不同试试直接ping外网地址。
> ```

> 运行container
>
> ```
> lxc-execute -n ubuntu -f config.ubuntu -- /bin/bash
> ```
>
> 通过ifconfig查看container网络
> 10.0.4.101，为config.ubuntu中设置的ip地址
> 为container增加默认路由
>
> ```
> route add  -net default gw 10.0.4.15
> #10.0.4.15为虚拟机的网桥的地址
> ```



> 设置DNS
>
> 在/etc/resolv.conf 中增加
> 		nameserver 10.0.4.15【理论上说，可以通过10.0.4.15解析域名，但是实地测试不生效】
> 		nameserver 8.8.8.8【增加google的DNS】
> ping baidu.com,OK
>
> 由于没有进行系统设备的设置，可能无法进行远程登录等动作，需要后续安装相关的资料再进行设置即可

2. 进行享受吧,但是发现有时可以，有时不行，不知道是不是virtualbox的问题
3. 试验了两天发现这种方式部署的container不是很稳定，偶发性的网络问题。
4. 下一步试试lxc-create

## LXC

> Linux软件容器（*Linux Containers*）的缩写，一种*操作系统层虚拟化（Operating system–level virtualization）技术*，为Linux内核容器功能的一个用户空间接口。它将应用软件系统打包成一个软件容器（Container），内含应用软件本身的代码，以及所需要的操作系统核心和库。

### lxc-create

> 之前经过debootstrap来创建container发现是可以的，但是发现总是不够稳定。下面将使用lxc-create试试

1. 通过lxc-create命令创建的container就很简单，详细如下：

   ```
     sudo lxc-create -t ubuntu -n ubuntu
     sudo lxc-start -n ubuntu -d
     sudo lxc-console -n ubuntu
   ```

  

2. 经过测试发现lxc-create创建的container也可以移动到指定的目录，使用如下方法：

   ```
     mv  /var/lib/lxc/ubuntu/ /DOING/LXC/
     lxc-start -f config -n ubuntu -d
   ```

     以此推理，采用 `sudo lxc-create -t ubuntu -n master --dir=/home/i/lxc/master`，也可以。

     相关的配置文件在$(pwd)/lxc-ok/

 

3. 基本的概念

   >一个container可以创建多个实例，通过lxc-start 来启动实例
   >如果container相同实例启动之后文件将相同，为了达到不同的实例不同的文件，则需要对不同的实例mount不同的目录

4. 配置IP地址
   
    > 默认启动的实例的IP地址为DHCP的
    >
    > 1. container实例配置
    >    	lxc.utsname=hadoop01【目前还没有发现有什么作用】
    >       	lxc.include=/DOING/LXC/ubuntu/config【公共的一些配置，网络参数】
    >       	lxc.network.hwaddr=00:16:3e:00:00:01【mac地址】
    >       	lxc.network.ipv4=10.0.3.101/24【静态IP地址，需要和lxcbr0在一个网段】
    >       	lxc.include=/DOING/LXC/ubuntu/fstab【需要mount的公共文件系统】
    >       	lxc.include=/DOING/LXC/hadoop01/fstab【需要mount的文件系统】
    > 2. ubuntu/config【此congfig文件在lxc-create的时候自动创建，后需要修改网络等参数】
    >    	lxc.include=/usr/share/lxc/config/ubuntu.common.conf【引用公共的ubuntu的配置文件】
    >       	lxc.rootfs =/DOING/LXC/ubuntu/rootfs.ubuntu【container目录】
    >       	lxc.network.type = veth
    >       	lxc.network.flags = up
    >       	lxc.network.link = lxcbr0【网桥名称】
    >       	lxc.network.name = eth0
    >       	lxc.network.mtu = 1500
    >       	lxc.network.ipv4.gateway = auto【自动获得网管】
    > 3. hadoop01/fstab
    >    	lxc.mount.entry=/TOOLS/hadoop 			home/ubuntu/hadoop 			none 	ro,bind 0 0
    >       	【源文件目录】					【需要挂载到的实例的目录，一定要存在】				
> 4. ubuntu/fstab
    >    	A. lxc.mount.entry=/TOOLS/jdk1.7.0_71_linux_x64 home/ubuntu/jdk1.7.0_71_linux_x64   none bind,ro 0 0

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



