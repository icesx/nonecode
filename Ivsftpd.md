###  yum install vsftpd
### 553错误的时候
	#setsebool -P allow_ftpd_full_access on
	#setsebool -P ftp_home_dir  on 
### 相关配置
	   vi /etc/vsftpd/vsftpd.conf	
		anonymous_enable=NO	【关闭匿名访问】
		userlist_deny=NO	【只允许userlist_file中的用户使用FTP】
		userlist_file=/etc/vsftpd/user_list	【允许使用ftp的本地账户】
		chroot_list_enable=NO	【】
		chroot_local_user=YES	【If chroot_local_user is YES, then this list becomes a list of users to NOT chroot()】
		allow_writeable_chroot=YES   【允许chroot的时候写文件】
		注：${home} 应该为755
		#被动模式	
		pasv_enable=YES
		pasv_min_port=6000
		pasv_max_port=6100 
	vi /etc/vsftpd/chroot_list
	
	vi /etc/vsftpd/user_list

### 防火墙设置
	iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
	iptables -A INPUT -p tcp  -m multiport --dport 20,21  -m state --state NEW -j ACCEPT
	如果有问题执行如下命令
	modprobe ip_conntrack_ftp
### 配置
	①当chroot_list_enable=YES，chroot_local_user=YES时，在/etc/vsftpd.chroot_list文件中列出的用户，可以切换到其他目录；未在文件中列出的用户，不能切换到其他目录。
	②当chroot_list_enable=YES，chroot_local_user=NO时，在/etc/vsftpd.chroot_list文件中列出的用户，不能切换到其他目录；未在文件中列出的用户，可以切换到其他目录。
	③当chroot_list_enable=NO，chroot_local_user=YES时，所有的用户均不能切换到其他目录。
	④当chroot_list_enable=NO，chroot_local_user=NO时，所有的用户均可以切换到其他目录。
### 匿名登录
	A、匿名登录的原理是使用ftp账户，ftp账户的信息可以通过cat /etc/passwd查看
	B、配置匿名需要在vsftpd.conf中增加如下配置
		userlist_enable=NO
		anonymous_enable=YES
		anon_upload_enable=yes
		anon_mkdir_write_enable=yes
		anon_other_write_enable=yes
		...


### 目前使用的配置为
	listen_port=61321
	userlist_enable=No
	userlist_deny=No
	chroot_list_enable=No
	chroot_local_user=YES
	allow_writeable_chroot=YES
	write_enable=YES

## ubuntu
### install
​	`sudo apt install vsftpd`

### config
	vi /etc/vsftpd.conf
## 端口转发

### 被动模式
1. 在vsftpd.conf 中增加数据端口限定
	pasv_min_port=61400
	pasv_max_port=61500 
2. 在防火墙中增加端口转发
-A PREROUTING -p tcp -m tcp --dport 61321 -j DNAT --to-destination 172.16.15.4
-A PREROUTING -p tcp -m tcp --dport 61400:61500 -j DNAT --to-destination 172.16.15.4