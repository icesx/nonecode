proxmox
==========
## install

+ 下载官方镜像
+ 刻录usb
+ 安装

```
dd bs=1M conv=fdatasync if=./proxmox-ve_*.iso of=/dev/XYZ
```

+ https://ip:8006/

### 修改默认22端口

proxmox默认使用22端口和root用户进行虚拟机的迁移工作，如果需要修改端口，需要按照如下修改

```
vi /etc/ssh/ssh_config
change 22 port to what you want
```



## 国内镜像

```
deb https://mirrors.ustc.edu.cn/proxmox/debian/pve buster pve-no-subscription
```

## 虚拟机安装

### iso
scp -P 60202 cn_windows_7_ultimate_x64_dvd_x15-66043.iso bjrdc-pmx@211.157.177.100:/home/bjrdc-pmx/ 
mv /home/bjrdc-pmx/cn_windows_7_ultimate_x64_dvd_x15-66043.iso /var/lib/vz/template/iso/

## backup and restore
### backup

+ 选中虚拟机
+ backup
+ 查看生成的备份文件lzo格式
+ 将其cp到本地


### restore
+ scp lzo 到服务器上
+ 在服务器上执行命令


```
qmrestore vzdump-qemu-101-2018_12_02-15_03_28.vma.lzo 101
```

### 跨节点cone

#### 手动

+ 在节点A上backup虚拟机
+ scp  /var/lib/vz/dump/***.lzo B:`pwd`
+ 在gpi上restore
+ 
#### 迁移

+ 
### 修改ip
+ [参考地址](https://forum.proxmox.com/threads/can-proxmox-server-ip-be-changed.43486/) 
+ change ip
```
vi /etc/network/interfaces
```
+ change /etc/hosts
+ /etc/pve/corosync.conf
```
      pvecm expect 1
       (this allows you to edit the file)
      nano /etc/pve/corosync.conf
      *update* the config_version number under totem section
      reboot
```

### windows2008
scsi驱动
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.141-1/virtio-win-0.1.141.iso
### 删除不用disk
	lvremove /dev/pve/vm-102-disk-1

### reset password

+ Boot into **grub**, select ***single user*** but **do not press enter**.

+ Press **e** to go into **edit** mode.

+ Scroll down to the **kernel** line you will boot from, it starts with "`linux /boot/vmlinuz-…….`"

+ Scroll to the end of that line and press **space key** once and type `init=/bin/bash`

+ Press **Ctrl X** to boot



```
  # Remount / as Read/Write 
  mount -rw -o remount /
  # Change the root account password with
  passwd
  # Change any other account password with
  passwd username
  # type new password, confirm and hit enter and then reboot.
```

# container

### 下载模板

```pveam update
pveam update
pveam available
pveam download local debian-10.0-standard_10.0-1_amd64.tar.gz
pveam list local
```

### 手动下载

```
wget http://download.proxmox.com/images/system/debian-10.0-standard_10.0-1_amd64.tar.gz
cp debian-10.0-standard_10.0-1_amd64.tar.gz /var/lib/vz/template/cache/
```

## 升级

### 5 升级到6

参考地址

https://pve.proxmox.com/wiki/Upgrade_from_5.x_to_6.0

### pve5to6

检查是否满足升级条件

```
pve5to6
```

### upgrade to Corosync 3 first

```
systemctl stop pve-ha-lrm
systemctl stop pve-ha-crm
echo "deb http://download.proxmox.com/debian/corosync-3/ stretch main" > /etc/apt/sources.list.d/corosync3.list
apt update
apt dist-upgrade
systemctl start pve-ha-lrm
systemctl start pve-ha-crm
```

### 升级到6

```
sed -i 's/stretch/buster/g' /etc/apt/sources.list
apt update
apt dist-upgrade
```

如果慢的话，可以使用aliyun和科大镜像加速

```
cat /etc/apt/sources.list
deb http://mirrors.aliyun.com/debian buster main contrib

deb http://mirrors.aliyun.com/debian buster-updates main contrib

# PVE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
#deb http://download.proxmox.com/debian/pve buster pve-no-subscription
deb https://mirrors.ustc.edu.cn/proxmox/debian/pve buster pve-no-subscription
# security updates
deb http://mirrors.aliyun.com buster/updates main contrib
```



## 集群

### 将已有机器添加到集群

1. 如果当前主机下有虚拟机，是无法将当前主机添加到集群中的，因为可能又vmid相同。
2. 备份本机的`/etc/pve/nodes/${node_name}`目录，其中有相关虚拟机和lxc的配置信息
3. 删除`/etc/pve/nodes/${node_name}`，重新刷新webui，会发现所有的虚拟机都没有了
4. 重新将当前机器添加的cluster，会发现所有的虚拟机又出来了。
5. 如果没有回来，把备份文件还原到`/etc/pve/nodes/`

### 撤离集群

```
systemctl stop pve-cluster.service
systemctl stop corosync.service
pmxcfs  -l
rm /etc/pve/corosync.conf
rm -rf /etc/corosync/*
killall pmxcfs
systemctl start pve-cluster.service
```

### 删除节点

```
pvecm delnode proxmox-node3
ls -l /etc/pve/nodes/
mv /etc/pve/nodes/proxmox-node3 /root/NodeName
```



## 用户

### 添加用户



## 注意事项

1. 

## 问题处理

### Host key verification failed.

参考地址 https://forum.proxmox.com/threads/host-key-verification-failed-when-migrate.41666/

```
ssh -o 'HostKeyAlias=bj3' root@192.168.86.44
```

### Proxmox TASK ERROR: VM is locked

ssh登陆Proxmox服务器，执行如下命令即可(ID：103)：
qm unlock 103

## 常用命令

1. 关闭虚拟机

   ```sh
   qm stop 100
   ```

2. 