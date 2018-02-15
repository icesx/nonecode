###ufw 相关的概念
	ufw是ubuntu的简单防火墙
###相关的命令
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
###端口转发
	A、打开linux的ip转发
		vi /etc/sysctl.cnf 
		net.ipv4.ip_forward=1
		【另外ufw也有一个类似的配置在/etc/ufw/sysctl.conf，最好这个也配置了】
	B、将ufw默认的转发功能打开
		vi /etc/default/ufw	
		#DEFAULT_FORWARD_POLICY="DROP"
		DEFAULT_FORWARD_POLICY="ACCEPT"
		【配置的时候出现多次配置不成功的情况，后来估计就是这个原因】
	C、vi /etc/ufw/befor.rulers
		 在*filter前面增加
		*nat
		:PREROUTING ACCEPT [0:0]
		:INPUT ACCEPT [0:0]
		:OUTPUT ACCEPT [0:0]
		:POSTROUTING ACCEPT [0:0]
		-A PREROUTING  -p tcp -m tcp --dport 1194 -j DNAT --to-destination 192.168.0.127
		-A POSTROUTING -j MASQUERADE
		COMMIT
	D、重启ufw
		ufw disable 
		ufw enable
		[似乎reload不行]
		或者重启操作系统
