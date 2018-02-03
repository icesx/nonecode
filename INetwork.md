###使用Linux建立私有网络的基本思路是
	A、选择一台机器作为路由器，在此机器上设置两个网卡，其中一个网卡接外网，一个网卡接内网
	B、打开ip_foward
	C、打开防火墙，设置转发规则，
	#iptables -t nat -A POSTROUTING -s 172.16.1.0/24 -o eth0 -j MASQUERADE
###nc -vuz 10.0.88.46 1195
	Connection to 10.0.88.46 1195 port [udp/*] succeeded!

