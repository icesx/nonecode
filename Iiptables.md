iptables
=========
### 基本概念
```
	iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
	iptables -A INPUT -p tcp  -m multiport --dport 20,21  -m state --state NEW -j ACCEPT
	NEW: 该包想要开始一个新的连接（重新连接或连接重定向）
	RELATED:该包是属于某个已经建立的连接所建立的新连接。如FTP的数据传输连接和控制连接之间就是RELATED关系。
	ESTABLISHED：该包属于某个已经建立的连接。
	INVALID:该包不匹配于任何连接，通常这些包被DROP。
```
### 删除规则

```
iptables -D INPUT 3  //删除input的第3条规则  
  
iptables -t nat -D POSTROUTING 1  //删除nat表中postrouting的第一条规则  
  
iptables -F INPUT   //清空 filter表INPUT所有规则  
  
iptables -F    //清空所有规则  
  
iptables -t nat -F POSTROUTING   //清空nat表POSTROUTING所有规则  
```



### 相关命令

 1. Delete all existing rules
```
	iptables -F
```
 2. Set default chain policies
 ```
	iptables -P INPUT DROP
	iptables -P FORWARD DROP
	iptables -P OUTPUT DROP
 ```
 3. list nat

    ```
    sudo iptables -t nat -L --line-numbers
    ```

    

 4. Block a specific ip-address
```
	BLOCK_THIS_IP="x.x.x.x"
	iptables -A INPUT -s "$BLOCK_THIS_IP" -j DROP
```
 4. Allow ALL incoming SSH
```
	iptables -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```
 5. Allow incoming SSH only from a sepcific network
```
	iptables -A INPUT -i eth0 -p tcp -s 192.168.200.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```
 6. Allow incoming HTTP
```
	iptables -A INPUT -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT

	 Allow incoming HTTPS
	iptables -A INPUT -i eth0 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```
 7. MultiPorts (Allow incoming SSH, HTTP, and HTTPS)
```
	iptables -A INPUT -i eth0 -p tcp -m multiport --dports 22,80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp -m multiport --sports 22,80,443 -m state --state ESTABLISHED -j ACCEPT
```
 8. Allow outgoing SSH
```
	iptables -A OUTPUT -o eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A INPUT -i eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```
 9. Allow outgoing SSH only to a specific network
```
	iptables -A OUTPUT -o eth0 -p tcp -d 192.168.101.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A INPUT -i eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```
 10. Allow outgoing HTTPS
```
	iptables -A OUTPUT -o eth0 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A INPUT -i eth0 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```
 11. Load balance incoming HTTPS traffic
```
	iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 0 -j DNAT --to-destination 192.168.1.101:443
	iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 1 -j DNAT --to-destination 192.168.1.102:443
	iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 2 -j DNAT --to-destination 192.168.1.103:443
```
 12. Ping from inside to outside
```
	iptables -A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT
	iptables -A INPUT -p icmp --icmp-type echo-reply -j ACCEPT
```
 13. Ping from outside to inside
```
	iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
	iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
```
 14. Allow loopback access
```
	iptables -A INPUT -i lo -j ACCEPT
	iptables -A OUTPUT -o lo -j ACCEPT
```
 15. Allow packets from internal network to reach external network.
```
	 if eth1 is connected to external network (internet)
	 if eth0 is connected to internal network (192.168.1.x)
	iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```
 16. Allow outbound DNS
```
	iptables -A OUTPUT -p udp -o eth0 --dport 53 -j ACCEPT
	iptables -A INPUT -p udp -i eth0 --sport 53 -j ACCEPT
```
 17. Allow NIS Connections
```
	 rpcinfo -p | grep ypbind ; This port is 853 and 850
	iptables -A INPUT -p tcp --dport 111 -j ACCEPT
	iptables -A INPUT -p udp --dport 111 -j ACCEPT
	iptables -A INPUT -p tcp --dport 853 -j ACCEPT
	iptables -A INPUT -p udp --dport 853 -j ACCEPT
	iptables -A INPUT -p tcp --dport 850 -j ACCEPT
	iptables -A INPUT -p udp --dport 850 -j ACCEPT
```
 18. Allow rsync from a specific network
```
	iptables -A INPUT -i eth0 -p tcp -s 192.168.101.0/24 --dport 873 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 873 -m state --state ESTABLISHED -j ACCEPT
```
 19. Allow MySQL connection only from a specific network
```
	iptables -A INPUT -i eth0 -p tcp -s 192.168.200.0/24 --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
```
 20. Allow Sendmail or Postfix
```
	iptables -A INPUT -i eth0 -p tcp --dport 25 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 25 -m state --state ESTABLISHED -j ACCEPT
```
 21. Allow IMAP and IMAPS
```
	iptables -A INPUT -i eth0 -p tcp --dport 143 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 143 -m state --state ESTABLISHED -j ACCEPT
	iptables -A INPUT -i eth0 -p tcp --dport 993 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 993 -m state --state ESTABLISHED -j ACCEPT
```
 22. Allow POP3 and POP3S
```
	iptables -A INPUT -i eth0 -p tcp --dport 110 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 110 -m state --state ESTABLISHED -j ACCEPT
	iptables -A INPUT -i eth0 -p tcp --dport 995 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 995 -m state --state ESTABLISHED -j ACCEPT
```
 23. Prevent DoS attack
```
	iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
```
 24. Port forwarding 422 to 22
```
	iptables -t nat -A PREROUTING -p tcp -d 192.168.102.37 --dport 422 -j DNAT --to 192.168.102.37:22
	iptables -A INPUT -i eth0 -p tcp --dport 422 -m state --state NEW,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -o eth0 -p tcp --sport 422 -m state --state ESTABLISHED -j ACCEPT
```
 25. Log dropped packets
```
	iptables -N LOGGING
	iptables -A INPUT -j LOGGING
	iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables Packet Dropped: " --log-level 7
	iptables -A LOGGING -j DROP
```

### 通过文件方式修改
```
	vi /etc/sysconfig/iptables
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 31817 -j ACCEPT
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 875 -j ACCEPT
	-A INPUT -p udp -m state --state NEW -m udp --dport 875 -j ACCEPT
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 892 -j ACCEPT
	-A INPUT -p udp -m state --state NEW -m udp --dport 892 -j ACCEPT
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 32803 -j ACCEPT
	-A INPUT -p udp -m state --state NEW -m udp --dport 32769 -j ACCEPT
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 2049 -j ACCEPT
	-A INPUT -p udp -m state --state NEW -m udp --dport 2049 -j ACCEPT
```
### save reload
	iptable-save > /etc/sysconfig/iptables
	iptable-restore < /etc/sysconfig/iptables

