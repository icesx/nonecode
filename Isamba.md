1、关闭seliux
disable selinux
2、修改samba的配置
	[homes]
	comment=Home Directories
	wirteable=yes
	
3、添加用户，一定要添加本地用户
#sambapasswd -a docker
zgjx@321
zgjx@321
