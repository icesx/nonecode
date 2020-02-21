proxmox
==========
## install

+ 下载官方镜像
+ 刻录usb
+ 安装
+ https://ip:8006/

## 虚拟机安装
### iso
​	scp scp -P 60202 cn_windows_7_ultimate_x64_dvd_x15-66043.iso bjrdc-pmx@211.157.177.100:/home/bjrdc-pmx/ 
​	mv /home/bjrdc-pmx/cn_windows_7_ultimate_x64_dvd_x15-66043.iso /var/lib/vz/template/iso/

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

+ 在节点A上backup虚拟机
+ scp  /var/lib/vz/dump/***.lzo B:`pwd`
+ 在gpi上restore
### 迁移
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