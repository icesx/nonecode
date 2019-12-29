### 安装

```
	#yum install bind
 	#vi /etc/named.conf
				listen-on port 53 { 127.0.0.1; };
				allow-query     { localhost; };
			修改为
				listen-on port 53 { any; };
				allow-query     { any; };
	#service named restart
```





