### usermod
`usermod -a -G wheel docker`
`logout`
###find
	find ./*  -mtime +7 -type f -a  -exec rm -f {} \;
	find . -exec cat {} \;|grep workSpace
###history
	home/.bach_profile
	export HISTTIMEFORMAT='%F %T '
	export HISTSIZE=45000
###tcpkill
	sudo netstat -ap | grep :<port_number>
	Also you can try this to close the socket connection
	tcpkill -i eth0 host xxx.xxx.xxx.xxx port yyyy
	Replace X with the IP address, and Y with the port number.
	example
	#tcpkill -i eth1 -9 host 183.129.145.18
###expect
 1. shell
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

 2. ssh yes
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
###cu
	#chown uucp /dev/ttyUSB0
	#sudo cu -l /dev/ttyUSB0 -s 115200
###timezone
	java 读取默认市区的时候，使用的可能是/etc/localtime。因为在centos7下出现的一个情况是，使用centos的命令tzselect。发现系统的时区修改了，但是java的时区未修改，解决办法是，修改/etc/localtime
	#ls -l /etc/localtime 
	lrwxrwxrwx. 1 root root 33 5月   5 09:41 /etc/localtime -> /usr/share/zoneinfo/Asia/Shanghai
### tar
	tar -cvf test.tgz test/ --exclude *.txt --exclude *.jpg
### nc
	$  nc -vz -u 10.1.0.100 53
	Connection to 10.1.0.100 53 port [udp/domain] succeeded!
	
### user home


	
