Security
=======

## 黑客工具

### hydra

```
hydra l test -p password.txt 192.168.87.138 ssh -t 5 -vv
```



### medusa



### aircrach-ng

破解[WEP](https://zh.wikipedia.org/wiki/有線等效加密)以及[WPA](https://zh.wikipedia.org/w/index.php?title=Wi-Fi_Protected_Access&action=edit&redlink=1)（[字典攻击](https://zh.wikipedia.org/w/index.php?title=字典攻击&action=edit&redlink=1)）密钥



### crunch

```
crunch 1 3 123 >123.txt
```

### hping3

```
hping3 -q -n -a 2.2.2.2 -S -s 53 --keep -p 445 --flood 192.168.73.138
```

(格式：hping3 -q -n -a 伪造源地址 -S -s 伪造源端口 --keep -p 攻击的目标端口 --flood 目标IP)

### burpsuite



## kali

渗透测试的操作系统

## openvas

### install

```
sudo apt-get install openvas
sudo gvm-setup
sudo gvm-feed-update
sudo gvm-start
```

### 登录

[https://localhost:9392](https://localhost:9392/)

### 远程

openvas 无法开启远程ip服务，故如果远程使用的话，就需要使用vnc或者rdp

```sh
sudo apt install xrdp 

```

> By default Xrdp uses the `/etc/ssl/private/ssl-cert-snakeoil.key` file that is readable only by members of the “ssl-cert” group. Run the following command to [add the `xrdp` user to the group](https://linuxize.com/post/how-to-add-user-to-group-in-linux/) :

```
sudo adduser xrdp ssl-cert  
sudo systemctl restart xrdp
sudo systemctl enable xrdp
sudo reboot
```

