proxmox
==========
##install
+ 下载官方镜像
+ 刻录usb
+ 安装
+ https://ip:8006/

/var/lib/vz/template/iso/
##backup and restore
###backip
+ 选中虚拟机
+ backup
+ 查看生成的备份文件lzo格式
+ 将其cp到本地
###restore
+ scp lzo 到服务器上
+ 在服务器上执行命令
```
qmrestore vzdump-qemu-101-2018_12_02-15_03_28.vma.lzo 101
```

###跨节点cone
+ 在节点A上backup虚拟机
+ scp  /var/lib/vz/dump/***.lzo B:`pwd`
+ 在gpi上restore

### 修改ip
+ [参考地址](https://forum.proxmox.com/threads/can-proxmox-server-ip-be-changed.43486/) 
+ change ip
```
	#vi /etc/network/interfaces
```
+ change /etc/hosts
+ /etc/pve/corosync.conf
```
      #pvecm expect 1
       (this allows you to edit the file)
      #nano /etc/pve/corosync.conf
      *update* the config_version number under totem section
      #reboot
```