Linux 
===========
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

ubuntu
-------------------

### 初始化安装

> 安装linux推荐最小化安装，之后再通过命令安装需要的组件

```sh
sudo apt install gcc make automake net-tools route zip unzip binutils
```
### 远程桌面

### vnc

0. 配置
`vino-preferences`
1. 开启vnc
`$/usr/lib/vino/vino-server`	
2. 开启权限	
$gsettings set org.gnome.Vino require-encryption false

### RDP



### 本地ISO软件源

> Ubuntu的软件源文件为/etc/apt/sources.list，我们可以先备份一下该文件，直接执行mv命令，这样就没有sources.list文件了。下面挂载ISO镜像，一般放了DVD会自动挂载，我们也可以手动挂载到/media/cdcrom
```sh
$mount /dev/cdrom /media/cdrom 
apt-cdrom -m -d=/media/cdrom add 
保留 /etc/apt/sources.list中的第一行
deb cdrom:[....]
现在执行
$apt-get update 
```
### 修改IP地址

#### ubuntu 16

```sh
$vi /etc/network/interfaces
auto eth0
iface eth0 inet static
address 192.168.31.10
netmask 255.255.255.0
gateway 192.168.31.1
```

修改NDS 

```sh
sudo systemctl enable dnsmasq
sudo vi /etc/dnsmasq.conf
	server=114.114.114.114
sudo systemctl start dnsmasq
```



#### ubuntu18

network配置发生变化，修改方式如下：
```yaml
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
netplan 相关命令

```sh
netplan apply
netplan generate
```

DNS

> ubuntu18以后，dns服务器是127.0.0.1:53，本地有一个dns服务器，如果需要增加远程的dns服务器，
>
> 1. 通过`resolvconf`来实现更新，（直接修改/etc/resolv.conf文件是会被系统自动覆盖的）相关方法如下**此方式不推荐**
>
> ```sh
> sudo apt install resolvconf
> cat > /etc/resolvconf/resolv.conf.d/head <<EOF
> nameserver 10.96.0.10
> EOF
> resolvconf -u
> ```
>
> 2. 使用systemd-resolved
>
>    修改`vi /etc/systemd/resolved.conf`
>
>    ```
>    [Resolve]
>    DNS=10.96.0.10
>    ```
>
>    重启服务
>
>    ```sh
>    sudo systemctl restart systemd-resolved
>    ```
>
>    
>
>    重建链接
>
>    ```sh
>    sudo rm -f /etc/resolv.conf
>    sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
>    ```
>
>    

### 多网卡配置

```
$ip link
```
### 带宽评估

准备server和client两台机器并分别安装ipref

```
sudo apt install iperf
```

在server上启动

```
iperf -s
```

在client上启动

```
iperf -c server
```

### 触摸板

1. 查看触摸板设备

   ```
   lsusb
   Bus 003 Device 003: ID 06cb:00f9 Synaptics, Inc.
   ```

2. 安装驱动

   ```
   sudo apt install xserver-xorg-input-synaptics
   ```


### desktop

注：desktop文件名必须与`startupWMclass`相同，这个属性用于窗口分类的，相同的进程会折叠到一起。

```
cat jetbrains-idea-ce.desktop 
```

```
[Desktop Entry]
Version=1.0
Type=Application
Name=IntelliJ IDEA Community Edition
Icon=/TOOLS/IDE/idea-IC/bin/idea.svg
Exec="/TOOLS/IDE/idea-IC/bin/idea.sh" %f
Comment=Capable and Ergonomic IDE for JVM
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-idea-ce

```

### nautilus

#### 隐藏bookmark

```
echo "enabled=False" > ~/.config/user-dirs.conf
vi ~/.config/user-dirs.dirs
```



### 修改文件权限

```
chmod -R 755 *
find . -type f -exec chmod -x {} \;
```



## systemd service

### systemd 基本命令

```sh
# 列出正在运行的 Unit
$ systemctl list-units

# 列出所有Unit，包括没有找到配置文件的或者启动失败的
$ systemctl list-units --all

# 列出所有没有运行的 Unit
$ systemctl list-units --all --state=inactive

# 列出所有加载失败的 Unit
$ systemctl list-units --failed

# 列出所有正在运行的、类型为 service 的 Unit
$ systemctl list-units --type=service

$ systemctl list-dependencies nginx.service
```

### service 文件格式

> sshd 的service文件
>
> ```
> systemctl cat sshd.service
> 
> # /usr/lib/systemd/system/sshd.service
> [Unit]
> Description=OpenSSH server daemon
> Documentation=man:sshd(8) man:sshd_config(5)
> After=network.target sshd-keygen.service
> Wants=sshd-keygen.service
> 
> [Service]
> Type=notify
> EnvironmentFile=/etc/sysconfig/sshd
> ExecStart=/usr/sbin/sshd -D $OPTIONS
> ExecReload=/bin/kill -HUP $MAINPID
> KillMode=process
> Restart=on-failure
> RestartSec=42s
> 
> [Install]
> WantedBy=multi-user.target
> ```
>
> 

```
[Unit]
Description=描述
Environment=环境变量或参数(系统环境变量此时无法使用)
After=network.target

[Service]
Type=forking
EnvironmentFile=所需环境变量文件或参数文件
ExecStart=启动命令(需指定全路径)
ExecStop=停止命令(需指定全路径)
User=以什么用户执行命令

[Install]
# 常用的 Target 有两个：一个是 multi-user.target，表示多用户命令行状态；另一个是 graphical.target，表示图形用户状态，它依赖于 multi-user.target
WantedBy=multi-user.target
```

### 增加自启动

1. 创建service文件

   ```sh
   cat harbor.service 
   [Unit]
   Description=Redis
   After=network.target
   
   [Service]
   ExecStart=/usr/local/bin/docker-compose -f /docker/harbor/docker-compose.yml start 
   
   [Install]
   WantedBy=multi-user.target
   ```

   

2. 服务质量harbor.service

   ```sh
   cp harbor/harbor.service /lib/systemd/system/
   ```

   

3. enable service

   ```sh
   systemctl enable harbor
   ```

   



## journalctl

查看完整日志

```
sudo journalctl -f -u kubelet
sudo journalctl -xe --no-pager
```

查看简短日志

```
sudo journalctl -xe
```





## ufw

POSTROUTING是源地址转换，要把你的内网地址转换成公网地址才能让你上网。

PREROUTING是目的地址转换，要把别人的公网IP换成你们内部的IP，才让访问到你们内部受防火墙保护的机器

### 相关的命令
```sh
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
```
### 端口转发
> 打开linux的ip转发

```
vi /etc/sysctl.cnf 
net.ipv4.ip_forward=1
```
【另外ufw也有一个类似的配置在/etc/ufw/sysctl.conf，最好这个也配置了】

> 将ufw默认的转发功能打开

```
vi /etc/default/ufw	
#DEFAULT_FORWARD_POLICY="DROP"
DEFAULT_FORWARD_POLICY="ACCEPT"
```
【配置的时候出现多次配置不成功的情况，后来估计就是这个原因】
​

> 端口转发【ufw没有端口转发的命令】

```
vi /etc/ufw/befor.rulers
在*filter前面增加
```



```
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
#将从 1194 端口来的包发送 到 172.16.10.15 的1194端口
-A PREROUTING  -p tcp -m tcp --dport 1194 -j DNAT --to-destination 172.16.10.15
#将从172.16.10.0/24来的包 通过ens20【外网网口】发送出去
-A POSTROUTING -s 172.16.10.0/24 -o ens20 -j MASQUERADE
COMMIT
		
```

> 本地端口转发

```
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A PREROUTING  -p tcp -m tcp --dport 443 -j DNAT --to-destination 172.31.238.220:80
-A POSTROUTING -s 172.31.238.0/24  -o eth0 -j MASQUERADE

COMMIT
```



> 重启ufw

```
ufw disable 
ufw enable
```
[似乎reload不行]或者重启操作系统

> log

```
ufw allow log 8400
tail -f /var/log/ufw.log
```
> delete

```
ufw status numbered
ufw delete 4
```


### 自启动

ubuntu18 后自启动进行了修改，采用systemd,需要做如下修改：

systemd默认读取/etc/systemd/system下的配置文件，该目录下的文件会链接/lib/systemd/system/下的文件。一般系统安装完/lib/systemd/system/下会有rc-local.service文件，即我们需要的配置文件。
链接过来：

```sh
sudo ln -fs /lib/systemd/system/rc-local.service /etc/systemd/system/rc-local.service
```


```sh
sudo vi  /etc/systemd/system/rc-local.service 
add content blow:
[Install]  
WantedBy=multi-user.target  
Alias=rc-local.service
```

sudo vim /etc/rc.local

```sh
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
```sh
sudo chmod 755 /etc/rc.local
```

## apt

```sh
sudo apt-get build-dep --download-only -o dir::cache=./ openssh-server
```



## fsck

Structure needs cleaning

```
fsck -AR -t ext4 -y
```

## Disk

### parted （超过2T分区）

​	超过2T分区后 fdisk不支持，需要用parted
```
$parted /dev/sdc
mklable gpt
unit TB
mkpart
mkpart primary 0.00TB 4.00TB
print
quit	
```

### 动态扫描硬盘

```
echo '- - -' >/sys/class/scsi_host/host0/scan
```

## NET


### TIME_WAIT

```
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_sack = 0
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_tw_reuse = 1
```
```
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
```
>net.ipv4.tcp_syncookies = 1 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_fin_timeout = 30 表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。
net.ipv4.tcp_keepalive_time = 1200 表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
net.ipv4.ip_local_port_range = 1024 65000 表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为1024到65000。
net.ipv4.tcp_max_syn_backlog = 8192 表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。
net.ipv4.tcp_max_tw_buckets = 5000表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。默认为180000，改为5000。对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于Squid，效果却不大。此项参数可以控制TIME_WAIT套接字的最大数量，避免Squid服务器被大量的TIME_WAIT套接字拖死。




### tcpkill

```
sudo netstat -ap | grep :<port_number>
```

Also you can try this to close the socket connection
```
tcpkill -i eth0 host xxx.xxx.xxx.xxx port yyyy
```
Replace X with the IP address, and Y with the port number.
```
tcpkill -i eth1 -9 host 183.129.145.18
```

### tcpdump

```
tcpdump tcp port 8088 -s 0 -v -w hsvod.pcap
```

### dns

```
sudo apt install resolvconf
cat <<EOF |sudo tee /etc/resolvconf/resolv.conf.d/head
nameserver 10.96.0.10
EOF
resolvconf -u
```



## SSH

### 禁止远程

```

```

## rsync

### 断点续传

```sh
rsync  -P -rsh=ssh -e 'ssh -p 60212' 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@192.168.86.46:/cloud/
```

后台执行

```sh
ctrl+z
[1]+  Stopped                 rsync -P -rsh=ssh -e 'ssh -p 60212' 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@192.168.86.46:/cloud/
root@bj3:~# bg %1
[1]+ rsync -P -rsh=ssh -e 'ssh -p 60212' 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@192.168.86.46:/cloud/ &
```

1为ctrl+z后显示的pid号

完整命令

```sh
bjrdc@fw:/cloud$ rsync -P -rsh=ssh 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@bjrdc7:/git 
sending incremental file list
1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar
        140.03M   0%   23.49MB/s    0:29:57  ^Z
[1]+  Stopped                 rsync -P -rsh=ssh 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@bjrdc7:/git
bjrdc@fw:/cloud$ bg %1
[1]+ rsync -P -rsh=ssh 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@bjrdc7:/git &
bjrdc@fw:/cloud$ jobs
[1]+  Running                 rsync -P -rsh=ssh 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@bjrdc7:/git &
bjrdc@fw:/cloud$ fg %1
rsync -P -rsh=ssh 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@bjrdc7:/git
        631.56M   1%   20.05MB/s    0:34:41 
```

## 前后台切换

### jobs

```
bjrdc@fw:/cloud$ jobs
[1]+  Running                 rsync -P -rsh=ssh 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@bjrdc7:/git &
```



### bg

```
bjrdc@fw:/cloud$ bg %1 
[1]+ rsync -P -rsh=ssh 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@bjrdc7:/git &
```



### fg

```
bjrdc@fw:/cloud$ fg %1
rsync -P -rsh=ssh 1607959169_2020_12_14_13.3.6-ee_gitlab_backup.tar bjrdc@bjrdc7:/git
        631.56M   1%   20.05MB/s    0:34:41 
```

## 其他命令

### install fonts

```
sudo cp -r ${fonts} /usr/share/fonts/
sudo fc-cache  -fv
```
### usermod

`usermod -a -G wheel docker`
`logout`

### sudoer

centos

```sh
usermod -a -G wheel docker
```

ubuntu

```
sudo usermod -a -G sudo username
```
### find
```
find ./*  -mtime +7 -type f -a  -exec rm -f {} \;
find . -exec cat {} \;|grep workSpace
```
### history
​	home/.bach_profile
```
export HISTTIMEFORMAT='%F %T '
export HISTSIZE=45000
```
### expect

 1. shell
 ```sh
 #/usr/bin/expect <<EOF
 set timeout 3000
 spawn scp ${zip_file} ${user}@${host}:${remote}/
 expect "password:"
 send "${password}\r"
 expect "$"
 spawn ssh ${user}@${host}
 expect "password:"
 send "${password}\r"
 expect "${user}@"
 send "${spark_commit} --master yarn-cluster --class  ${main_class} /home/docker/mrs/${jar}\r"
 expect EOF  
 ```
 2. ssh yes

 ```sh
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
```sh
chown uucp /dev/ttyUSB0
sudo cu -l /dev/ttyUSB0 -s 115200
```
### timezone
	java 读取默认市区的时候，使用的可能是/etc/localtime。因为在centos7下出现的一个情况是，使用centos的命令tzselect。发现系统的时区修改了，但是java的时区未修改，解决办法是，修改/etc/localtime
	#ls -l /etc/localtime 
	lrwxrwxrwx. 1 root root 33 5月   5 09:41 /etc/localtime -> /usr/share/zoneinfo/Asia/Shanghai
### tar
	tar -cvf test.tgz test/ --exclude *.txt --exclude *.jpg
### nc

udp

	$  nc -vz -u 10.1.0.100 53
	Connection to 10.1.0.100 53 port [udp/domain] succeeded!

### user home
	usermod -m -d /newhome/username username

### subString

### lsblk

```sh
lsblk -l
```

### journalctl 

```
journalctl -ax --no-page
```



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

```
ip link show
```


### 串口

```
cu -l ttyAMA0 -s 115200
```

### bash

1. scp

   ```sh
   #!/bin/bash
   set -x
   _file=$(pwd $0)/$(dirname $0)
   project_path=$(dirname $_file)
   scp -P 22 $project_path/target/board-gateway-0.0.1-SNAPSHOT-package.jar bjrdc@bjrdc23:/home/bjrdc/push
   
   ```
   
2. 通过如下命令可以实现for 循环 按照回车进行 换行，而不是 空格
    [参考地址](https://www.cnblogs.com/cocowool/archive/2013/01/15/2861904.html) 

  ```
  	 IFS=$(echo -en "\n\b")
          echo -en $IFS
  ```


## Samba
1、关闭seliux
disable selinux
2、修改samba的配置
	[homes]
	comment=Home Directories
	wirteable=yes
	
3、添加用户，一定要添加本地用户

```sh
sambapasswd -a docker
xxxx
xxxx
```



## DNS

```sh
yum install bind
vi /etc/named.conf
				listen-on port 53 { 127.0.0.1; };
				allow-query     { localhost; };
			修改为
				listen-on port 53 { any; };
				allow-query     { any; };
service named restart
```



## vim

相信很多人在`vim`里粘贴带有缩进的内容都会遇到下面这种炫酷的效果

这个其实不是什么 bug。而是自动缩进`autoindent`搞的鬼，那我们也不能关了自动缩进啊。其实还有种解决方法，在粘贴前输入下面的命令：

```
:set paste
i
:set nopaste
：wq！
```

可以设置快捷键

Add this to your `~/.vimrc` and you will only have to press **F2** before and after pasting:

```
set pastetoggle=<F2>
```

## sendmail

1. install

   ```sh
   sudo apt install postfix mailutils
   ```

2. 发送邮件

   ```sh
   echo "test message42..." | mail -s 'nagios notification' 13824365716@139.com -r 'nagios@bjrdc51.xjgz.com
   echo "test message42..." | mailx -s 'nagios notification' 13824365716@139.com -r 'nagios@bjrdc51.xjgz.com
   -r : from
   ```

   

## cat EOF

```
cat >> /root/test.txt <<EOF
Hello!
My site is www.361way.com
My site is www.91it.org
Test for cat and EOF!
EOF
```



## nfs

```sh
sudo apt install nfs-kernel-server
cat >>/etc/exports <<EOF
/cloud/nfs-root *(rw,sync,no_subtree_check)
EOF
sudo chmod 777 /cloud/nfs-root
sudo systemctl restart nfs-kernel-server
sudo showmount -e localhost
sudo apt install nfs-common
sudo mount bjrdc216:/cloud/nfs-root temp/
```

## sed

```sh
echo xx,yy,zz >sed.text
echo 1,2,yy,x>> sed.text
sed -e "/xx/ s/yy/../g" sed.text 
xx,..,zz
1,2,yy,x

```

## ss

You can forcibly close sockets with `ss` command; the `ss` command is a tool used to dump socket statistics and displays information in similar fashion (although simpler and faster) to netstat.

To kill any socket in CLOSE_WAIT state, run this (as root)

```
$ ss --tcp state CLOSE-WAIT --kill
```

## curl

```
echo bjrdc:xxx|base64
YmplZGM6eHh4Cg==
```

```
curl -H "Authorization:Basic YmplZGM6eHh4Cg==" http://rabbit-stateful-0.rabbit-stateful-headless.bjrdc-dev.svc.cluster.local:15672/api/overview
```

```
curl -i -u guest:guest http://localhost:15672/api/vhosts
```

## patch

1. 下载需要的patch文件如

   ```sh
   wget https://issues.apache.org/jira/secure/attachment/12962642/HADOOP-16167.004.patch
   ```

2. 补丁操作

   ```sh
   patch software/hadoop-3.0.3/libexec/hadoop-functions.sh <HADOOP-16167.004.patch
   ```

   

## csvkit



```shll
sudo apt install csvkit
```

```
for i in `find .|grep csv|grep -v ok`; do csvsql --db "mysql://hav:hav321@bjrdc37/hav"  --insert $i;echo $i; done
```

## date

随机生成时间

```
for i in {0..10000}; do date -d "$((RANDOM%1+2020))-$((RANDOM%6+1))-$((RANDOM%28+1)) $((RANDOM%22+1)):$((RANDOM%59+1)):$((RANDOM%59+1))" '+%Y-%m-%d,%H:%M:%S'; done >> time
```

随机抽取行

```
shuf -n 100 Chinese_Names_Corpus_Gender（120W）.txt |awk -F ',' '{print $1}' > name.txt
```

