Linux 
===========
ubuntu
-------------------
### 初始化安装
```
sudo apt install gcc make automake net-tools route zip unzip 
```
### 远程桌面

0. 配置
`vino-preferences`
1. 开启vnc
`$/usr/lib/vino/vino-server`	
2. 开启权限	
$gsettings set org.gnome.Vino require-encryption false
### 本地ISO软件源
	Ubuntu的软件源文件为/etc/apt/sources.list，我们可以先备份一下该文件，直接执行mv命令，这样就没有sources.list文件了。下面挂载ISO镜像，一般放了DVD会自动挂载，我们也可以手动挂载到/media/cdcrom
```
	$mount /dev/cdrom /media/cdrom 
	apt-cdrom -m -d=/media/cdrom add 
	保留 /etc/apt/sources.list中的第一行
	deb cdrom:[....]
	现在执行
	$apt-get update 
```
### 修改IP地址

#### ubuntu 16

```
$vi /etc/network/interfaces
auto eth0
iface eth0 inet static
address 192.168.31.10
netmask 255.255.255.0
gateway 192.168.31.1
```

修改NDS 

```
sudo systemctl enable dnsmasq
sudo vi /etc/dnsmasq.conf
	server=114.114.114.114
sudo systemctl start dnsmasq
```



#### ubuntu18

	network配置发生变化，修改方式如下：
```
solar@solarsystem:~$ cat /etc/netplan/01-netcfg.yaml 
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
      eno1:
        dhcp4: no
        dhcp6: no
        addresses: [192.168.31.101/24]
        gateway4: 192.168.31.1
        nameservers:
              addresses: [192.168.31.1]
```
### 多网卡配置
```
	$ip link
```
## ufw

POSTROUTING是源地址转换，要把你的内网地址转换成公网地址才能让你上网。

PREROUTING是目的地址转换，要把别人的公网IP换成你们内部的IP，才让访问到你们内部受防火墙保护的机器

### ufw 相关的概念
​	ufw是ubuntu的简单防火墙
### 相关的命令
```
	   sudo ufw status
​	   ufw disable 
​	   ufw enable 
​	   ufw  status
​	   ufw  allow out 80,8080/tcp
​	   ufw allow out 53
​	   sudo ufw allow out to any port 53
​	   ufw default allow 
​	   ufw default deny 
​	   ufw status verbose 
​	   ufw status numbered 
​	   ufw delete 11
```
### 端口转发
​	A、打开linux的ip转发
```
vi /etc/sysctl.cnf 
net.ipv4.ip_forward=1
```
​		【另外ufw也有一个类似的配置在/etc/ufw/sysctl.conf，最好这个也配置了】
​	B、将ufw默认的转发功能打开

```
vi /etc/default/ufw	
#DEFAULT_FORWARD_POLICY="DROP"
DEFAULT_FORWARD_POLICY="ACCEPT"
```
​		【配置的时候出现多次配置不成功的情况，后来估计就是这个原因】
​	C、端口转发【ufw没有端口转发的命令】
```
		vi /etc/ufw/befor.rulers
​		 在*filter前面增加
​		*nat
​		:PREROUTING ACCEPT [0:0]
​		:INPUT ACCEPT [0:0]
​		:OUTPUT ACCEPT [0:0]
​		:POSTROUTING ACCEPT [0:0]
​		#将从 1194 端口来的包发送 到 172.16.10.15 的1194端口
​		-A PREROUTING  -p tcp -m tcp --dport 1194 -j DNAT --to-destination 172.16.10.15
​		#将从172.16.10.0/24来的包 通过ens20【外网网口】发送出去
​		-A POSTROUTING -s 172.16.10.0/24 -o ens20 -j MASQUERADE
​		COMMIT
​		
```
	D、重启ufw
```
		ufw disable 
		ufw enable
```
[似乎reload不行]或者重启操作系统
E、log
```
		ufw allow log 8400
		tail -f /var/log/ufw.log
```
F、delete
```
ufw status numbered
ufw delete 4
```
### 自启动
ubuntu18 后自启动进行了修改，采用systemd,需要做如下修改
```
1. sudo vim /etc/systemd/system/rc-local.service
2.  add code to rc-local.service
```
```
[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local
 
[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99
 
[Install]
WantedBy=multi-user.target
```
3. sudo vim /etc/rc.local
```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.

# By default this script does nothing.

exit 0
```
3. run this command
```
sudo chmod 755 /etc/rc.local
sudo systemctl enable rc-local
sudo systemctl start rc-local.service
sudo systemctl status rc-local.service

```

Centos
---------------------

### after mini install

```
yum update
yum install net-tools gcc make automake	
```

### zip 乱码

```
sudo apt install unar
$ lsar filename.zip
$ unar filename.zip
```
### dns
```

For static IP situations, the Ubuntu Server Guide says to change the file /etc/network/interfaces, which may look like this:

iface eth0 inet static
address 192.168.3.3
netmask 255.255.255.0
gateway 192.168.3.1
dns-search example.com
dns-nameservers 192.168.3.45 192.168.8.10
```
### parted （超过2T分区）
​	超过2T分区后 fdisk不支持，需要用parted
```
	$parted /dev/sdc
​	mklable gpt
​	unit TB
​	mkpart
​	mkpart primary 0.00TB 4.00TB
​	print
​	quit
​	
```

### TIME_WAIT

```
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_sack = 0
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_tw_recycle = 1
```
### install fonts
```
sudo cp -r ${fonts} /usr/share/fonts/
sudo fc-cache  -fv
```
### usermod

`usermod -a -G wheel docker`
`logout`

### sudoer
​	sudo usermod -a -G sudo username

### find
​	find ./*  -mtime +7 -type f -a  -exec rm -f {} \;
​	find . -exec cat {} \;|grep workSpace
### history
​	home/.bach_profile
​	export HISTTIMEFORMAT='%F %T '
​	export HISTSIZE=45000
### tcpkill
​	sudo netstat -ap | grep :<port_number>
​	Also you can try this to close the socket connection
​	tcpkill -i eth0 host xxx.xxx.xxx.xxx port yyyy
​	Replace X with the IP address, and Y with the port number.
​	example
​	#tcpkill -i eth1 -9 host 183.129.145.18

### expect

 1. shell
 ```
	#/usr/bin/expect <<EOF
	set timeout 3000
	spawn scp ${zip_file} ${user}@${host}:${remote}/
	expect "password:"
	send "${password}\r"
	#expect "$"
	#spawn ssh ${user}@${host}
	#expect "password:"
	#send "${password}\r"
	#expect "${user}@"
	#send "${spark_commit} --master yarn-cluster --class  ${main_class}  /home/docker/mrs/${jar}\r"
	expect EOF  
 ```
 2. ssh yes
 ```
	#!/usr/bin/expect
	set user [lindex $argv 0] 
	set host [lindex $argv 1] 
	set password root
	set timeout -1
	spawn ssh $user@$host
	expect {  
	  "*yes/no" { send "yes\r"}
	  "$user@" {send "exit\r"}
	}
	expect "$user@"
	send "exit\r"
	interact
 ```
### cu
```
	#chown uucp /dev/ttyUSB0
	#sudo cu -l /dev/ttyUSB0 -s 115200
```
### timezone
	java 读取默认市区的时候，使用的可能是/etc/localtime。因为在centos7下出现的一个情况是，使用centos的命令tzselect。发现系统的时区修改了，但是java的时区未修改，解决办法是，修改/etc/localtime
	#ls -l /etc/localtime 
	lrwxrwxrwx. 1 root root 33 5月   5 09:41 /etc/localtime -> /usr/share/zoneinfo/Asia/Shanghai
### tar
	tar -cvf test.tgz test/ --exclude *.txt --exclude *.jpg
### nc
	$  nc -vz -u 10.1.0.100 53
	Connection to 10.1.0.100 53 port [udp/domain] succeeded!

### user home
	usermod -m -d /newhome/username username

### subString

## ulimit 

1. /etc/security/limits.conf
​	$echo "* - nofile 102400" >> /etc/security/limits.conf/etc/pam.d/login

​	session required /lib/security/pam_limits.so 

2. /etc/sysctl.cfg

	fs.file-max=102400
3. /etc/ssh/sshd_config

​	UsePAM yes

### mount
	/etc/fstab
	mount -a


### rc.local

#### 指定用户启动
```
su - docker -c "/usr/local/mysql/bin/mysqld_safe --user=mysql&"
```
### network
#### 显示网卡
​	ip link show
### 串口
​	cu -l ttyAMA0 -s 115200

### bash

1. scp

   ```
   #!/bin/bash
   set -x
   _file=$(pwd $0)/$(dirname $0)
   project_path=$(dirname $_file)
   scp -P 22 $project_path/target/board-gateway-0.0.1-SNAPSHOT-package.jar bjrdc@bjrdc23:/home/bjrdc/push
   
   ```
2. 通过如下命令可以实现for 循环 按照回车进行 换行，而不是 空格
[参考地址](https://www.cnblogs.com/cocowool/archive/2013/01/15/2861904.html) 
```cu
	 IFS=$(echo -en "\n\b")
        echo -en $IFS
```

