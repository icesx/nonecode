###server
	#sudo apt-get install ntp
	#sudo service ntp start
	#vi /etc/ntp.conf
		
		server 127.127.1.0
		fudge 127.127.1.0 stratum 8	
	#ntpq -p
###客户端
	#sudo ntpdate -d hadoop-xj173
	#sudo ntpdate -u hadoop-xj173【更新本地时间】
###exiting nameservercannot be used
	为ntpserver 增加hostname
###时区
	#date -R
### no server suitabl for synchronization found
	chenge to ip not use hostname
###on server 
	watch ntpq -p
