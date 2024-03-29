openvpn
========
## 安装
### 操作系统配置

```
cat >> /etc/sysctl.conf <<EOF
	net.ipv4.ip_forward = 1
EOF
sysctl -p
```



### centos

​	centos7上yum里没有openvpn的源，需要安装readhead的源，使用如下命令

```
yum install epel-release
yum install openvpn easy-rsa -y
yum install 
```

### 关闭 selinux 

```
setenforce 0 
vi /etc/sysconfig/selinux
```

### ubuntu

#### apt

```
sudo apt install openvpn easy-rsa -y
```

#### wget

```sh
wget http://ftp.us.debian.org/debian/pool/main/e/easy-rsa/easy-rsa_2.2.2-1_all.deb
sudo dpkg -i reasy-rsa_2.2.2-1_all.deb
```


## 配置
### 参数配置

```
cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
gunzip server.conf.gz
vi /etc/openvpn/server.conf
		dh dh2048.pem
		push "redirect-gateway def1 bypass-dhcp"【这行如果加上的话，所有的网关走的是openvpn提供的，可以用了翻墙，但是如果加上了，会影响客户端的网速，甚至出现网络不通的情况】
		push "route 10.0.88.0 255.255.255.0"	【为客户端增加一条指向10.0.88.0/24的路由】
		push "route 172.16.1.0 255.255.255.0"	【为客户端增加一条指向172.16.1.0/24的路由，用于方位内部子网】
		push "dhcp-option DNS 8.8.8.8"
		push "dhcp-option DNS 8.8.4.4"
		user nobody
		group nobody
		#tls-auth ta.key 0 # This file is secret
```


### key on easy-rsa2
```
mkdir -p /etc/openvpn/easy-rsa/keys
cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa
```

### key on easy-rsa3

```sh
wget https://github.com/OpenVPN/easy-rsa/archive/master.zip
unzip master.zip
mv easy-rsa-master easy-rsa
cd /etc/openvpn/easy-rsa/easyrsa3/	
./easyrsa init-pki
./easyrsa gen-req server nopass
./easyrsa sign server server #注，这里前一个server是命令表示注册的是server端，后一个server是可以自行定义的名字，
./easyrsa gen-dh
./easyrsa gen-req client
cp pki/private/server.key /etc/openvpn/             
cp pki/issued/server.crt /etc/openvpn/                          
cp pki/dh.pem /etc/openvpn/
```

### vars

**一定要作**

```
vi /etc/openvpn/easy-rsa/vars
​		export KEY_NAME="server"
​		【KEY_NAME: You should enter server here; you could enter something else, but then you would also have to update the configuration files that reference server.key and server.crt】
​		【KEY_CN: Enter the domain or subdomain that resolves to your server】
​		export KEY_CN=openvpn.xjgz
​		export KEY_ALTNAMES="xjgz.cn"
```

### 生成证书

```
cp /etc/openvpn/easy-rsa/openssl-1.0.0.cnf /etc/openvpn/easy-rsa/openssl.cnf
cd /etc/openvpn/easy-rsa
source ./vars
./clean-all
./build-ca
./build-key-server server
./build-dh
cd /etc/openvpn/easy-rsa/keys
cp dh2048.pem ca.crt server.crt server.key /etc/openvpn
cd /etc/openvpn/easy-rsa
./build-key client	【client是你需要授权的账户，可以多次执行该命令来生成多个客户端的账户授权文件，如./build-key langk】
```



### 服务配置

```
systemctl -f enable openvpn@server.service
systemctl start openvpn@server.service
如果如上命令不能执行，使用如下命令：
openvpn --config /etc/openvpn/server.cnf
ifconfig p4p1:0 172.16.1.200 up	【增加一个虚拟网卡，用于链接到172.16.1.0/24网段】
```


### 客户端配置

​	A、将如下三个文件发布给客户端机器

> /etc/openvpn/easy-rsa/keys/ca.crt
> /etc/openvpn/easy-rsa/keys/client.crt
> /etc/openvpn/easy-rsa/keys/client.key


```
cat >/etc/openvpn/client.ovpn << EOF
client
dev tun
proto udp
remote ${服务ip地址} 1194
resolv-retry infinite
nobind
persist-key
persist-tun
comp-lzo
verb 3
ca /path/to/ca.crt
cert /path/to/client.crt
key /path/to/client.key
EOF
```
C、客户端启动
```
openvpn --config /etc/openvpn/client.ovpn
ping 10.8.0.1	【应该可以ping通】
ping 10.0.88.40	【应该ping不通】
```
### 防火墙配置
#### centos7 
```
systemctl stop firewalld
systemctl mask firewalld
yum install iptables-services -y
systemctl enable iptables
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o p4p1 -j MASQUERADE【将来源于10.8.0.0/24的网段转发到 p4p1网卡——即讲客户端的请求转发到10.0.88.0/24这个网卡上】
ping 10.0.88.40	【应该ping通】
```
记得在10.0.88.70上增加指向172.16.1.0/24的路由
```
route add -net 172.16.1.0/24 gw 10.0.88.40
#[新疆]
iptables -t nat -A POSTROUTING -d 192.168.0.0/24 -o eth0 -j MASQUERADE
iptables -t nat -A POSTROUTING -d 172.16.1.0/24 -o eth1 -j MASQUERADE
```
#### ubuntu 
将ufw默认的转发功能打开

```
vi /etc/sysctl.conf
net.ipv4.ip_forward=1
sudo systctl -p
```



```
vi /etc/default/ufw	
```
```
	#DEFAULT_FORWARD_POLICY="DROP"
	DEFAULT_FORWARD_POLICY="ACCEPT"
```
```
vi /etc/ufw/befor.rules
```
```
*nat
:POSTROUTING ACCEPT [0:0]
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
```
```
另外一种配置方法
-A POSTROUTING -d 172.16.15.0/24 -o ens19 -j MASQUERADE
# 访问外网
-A POSTROUTING -j MASQUERADE
```
```
ufw enable
ufw start
ufw reload
```
### 多客户端
```
source ./vars
./build-key langk
./build-key jinwl
```
### 注销账户
```sh
source ./vars
./revoke-full client1
vim /etc/openvpn/server.conf
crl-verify /etc/openvpn/easy-rsa/keys/crl.pem
#Reload the OpenVPN server to activate the revoke setting onle once.
service openvpn@server restart
```
> 按照此种方式进行账户的吊销后,30天后所有的客户端即连接不上了。原因是在生成的crl.pem文件中有一个
>
> ```
> Next Update: May 30 02:26:22 2020 GMT
> ```
>
> 所以在30天后，需要重新吊销证书
>
> 可以将30天修改成10年，修改方式是
>
> ```
> vi /etc/openvpn/easy-rsa/openssl.conf
> default_crl_days= 3650                  # how long before next CRL
> ```
>
> 之后重新吊销任何一个证书，即可更新crl.pem文件

### 查看有效用户



```sh
cd /etc/openvpn/easy-rsa/keys/
cat index.txt | grep ^V | awk -F "/" '{print $7, $8}'
```

```
V       300422073202Z           05      unknown /C=US/ST=CA/L=SanFrancisco/O=Fort-Funston/OU=MyOrganizationalUnit/CN=wanwt/name=server/emailAddress=me@myhost.mydomain
R       300507002139Z   211011122859Z   06      unknown /C=US/ST=CA/L=SanFrancisco/O=Fort-Funston/OU=MyOrganizationalUnit/CN=fugy/name=server/emailAddress=me@myhost.mydomain
```

V开头的是有效用户，R开头的是吊销用户



### 问题处理

1. Failed to start OpenVPN Robust And Highly Flexible Tunneling Application On server.
	注释掉
	
	`#tls-auth ta.key 0 # This file is secret`
	
	
	
2. [OpenSSL Error messages: error:1416F086:SSL routines:tls_process_server_certificate:certificate verify failed](https://stackoverflow.com/questions/57358047/openssl-error-messages-error1416f086ssl-routinestls-process-server-certifica)

3. 如果无法ping通vpn的网关可打开`redirect-gateway`