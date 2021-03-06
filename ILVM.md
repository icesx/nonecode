LVM on Linux
====

### 新建分区

```sh
fdisk /dev/sdb
partprobe
```
### 扩展lvm
```sh
vgcreate cdc /dev/sdb
lvcreate -L 4.5T -n cloud cdc
lvcreate -l 100%FREE -n cloud cdc	lvreduce -L 500G /dev/centos_dell30/home
lvextend -L +1T /dev/centos_dell30/root
lvextend -l +100%FREE /dev/centos_dell30/root
vgdisplay 
lvdisplay 
mkfs.ext4 /dev/cdc/cloud 
mkdir /cloud
mount /dev/cdc/cloud /cloud/
resize2fs  /dev/testvg/testlv
xfs_growfs /dev/mapper/VolGroup00-LogVol05
```
### 关于扩容
> 在vmware上调整/dev/sdb的磁盘大小,创建新的分区

```sh
fdisk /dev/sdb
```
> 扩展分区

```bash
vgextend cdc /dev/sdb2
lvextend -l +100%FREE /dev/mapper/cdc-cloud
resize2fs /dev/mapper/cdc-cloud
```
### rescan disk
新加硬盘后通过如下命令识别到
```bash
ls /sys/class/scsi_host
echo "- - -" > /sys/class/scsi_host/host#/scan
```
### rescan disksize
增加硬盘容量后，通过如下命令识别到
```bash
echo '- - -' > /sys/class/scsi_disk/2\:0\:1\:0/device/rescan
#2:0:1:0为scasi盘的盘符
```
### 超过2T分区
超过2T分区后 fdisk不支持，需要用parted
```bash
parted /dev/sdc
	mklable gpt
	unit TB
	mkpart
	mkpart primary 0.00TB 4.00TB
	print
	quit
```