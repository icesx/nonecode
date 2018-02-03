###远程桌面
	0、配置
	#vino-preferences
	1、开启vnc
	#/usr/lib/vino/vino-server	
	2、开启权限	
	#gsettings set org.gnome.Vino require-encryption false
###本地ISO软件源
	Ubuntu的软件源文件为/etc/apt/sources.list，我们可以先备份一下该文件，直接执行mv命令，这样就没有sources.list文件了。下面挂载ISO镜像，一般放了DVD会自动挂载，我们也可以手动挂载到/media/cdcrom
	#mount /dev/cdrom /media/cdrom 
	apt-cdrom -m -d=/media/cdrom add 
	保留 /etc/apt/sources.list中的第一行
	deb cdrom:[....]
	现在执行
	#apt-get update 
###修改IP地址
	#vi /etc/network/interfaces
	#ip addr flush dev ens160
	#service networking restart
###多网卡配置
	#ip link
