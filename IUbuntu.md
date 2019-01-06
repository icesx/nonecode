###远程桌面
0. 配置
vino-preferences
1. 开启vnc
$/usr/lib/vino/vino-server	
2. 开启权限	
$gsettings set org.gnome.Vino require-encryption false
###本地ISO软件源
	Ubuntu的软件源文件为/etc/apt/sources.list，我们可以先备份一下该文件，直接执行mv命令，这样就没有sources.list文件了。下面挂载ISO镜像，一般放了DVD会自动挂载，我们也可以手动挂载到/media/cdcrom
	$mount /dev/cdrom /media/cdrom 
	apt-cdrom -m -d=/media/cdrom add 
	保留 /etc/apt/sources.list中的第一行
	deb cdrom:[....]
	现在执行
	$apt-get update 
###修改IP地址
	$vi /etc/network/interfaces
	$ip addr flush dev ens160
	$service networking restart
####ubuntu18
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
###多网卡配置
	$ip link
##ufw

###ufw 相关的概念
	ufw是ubuntu的简单防火墙
###相关的命令
	   sudo ufw status
	   ufw disable 
	   ufw enable 
	   ufw  status
	   ufw  allow out 80,8080/tcp
	   ufw allow out 53
	   sudo ufw allow out to any port 53
	   ufw default allow 
	   ufw default deny 
	   ufw status verbose 
	   ufw status numbered 
	   ufw delete 11
###端口转发
	A、打开linux的ip转发
		vi /etc/sysctl.cnf 
		net.ipv4.ip_forward=1
		【另外ufw也有一个类似的配置在/etc/ufw/sysctl.conf，最好这个也配置了】
	B、将ufw默认的转发功能打开
		vi /etc/default/ufw	
		#DEFAULT_FORWARD_POLICY="DROP"
		DEFAULT_FORWARD_POLICY="ACCEPT"
		【配置的时候出现多次配置不成功的情况，后来估计就是这个原因】
	C、端口转发【ufw没有端口转发的命令】
		vi /etc/ufw/befor.rulers
		 在*filter前面增加
		*nat
		:PREROUTING ACCEPT [0:0]
		:INPUT ACCEPT [0:0]
		:OUTPUT ACCEPT [0:0]
		:POSTROUTING ACCEPT [0:0]
		-A PREROUTING  -p tcp -m tcp --dport 1194 -j DNAT --to-destination 192.168.0.127
		-A POSTROUTING -j MASQUERADE
		COMMIT
	D、重启ufw
		ufw disable 
		ufw enable
		[似乎reload不行]
		或者重启操作系统
	E、log
		ufw allow log 8400
		tail -f /var/log/ufw.log
	F、delete
		ufw status numbered
		ufw delete 4
### zip 乱码
```
sudo apt install unar
$ lsar filename.zip
$ unar filename.zip
```
###ulimit

Modify /etc/systemd/user.conf and /etc/systemd/system.conf with the following line (this takes care of graphical login):

	DefaultLimitNOFILE=65535
Modify /etc/security/limits.conf with the following lines (this takes care of non-GUI login):

	* hard nofile 65535
	* soft nofile 65535
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
###超过2T分区
	超过2T分区后 fdisk不支持，需要用parted
	$parted /dev/sdc
	mklable gpt
	unit TB
	mkpart
	mkpart primary 0.00TB 4.00TB
	print
	quit
	
### shell for 
通过如下命令可以实现for 循环 按照回车进行 换行，而不是 空格
[参考地址](https://www.cnblogs.com/cocowool/archive/2013/01/15/2861904.html) 
```
	 IFS=$(echo -en "\n\b")
        echo -en $IFS
```
###TIME_WAIT
```
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_sack = 0
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_tw_recycle = 1
```