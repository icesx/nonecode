### server

server bjrdc-ntp

```
sudo apt-get install ntp
sudo systemctl enable ntp
sudo systemctl start ntp
```
### 客户端
```
sudo apt-get install ntp
```
vi /etc/ntp.cnf,and server

```
server bjrdc5 prefer iburst
```

start ntp 

```
sudo systemctl enable ntp
sudo systemctl start ntp
```

查看ntp状态

```
ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 0.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 1.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 2.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 3.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 ntp.ubuntu.com  .POOL.          16 p    -   64    0    0.000    0.000   0.000
*bjrdc-ntp       203.107.6.88     3 u   48   64   17    0.486   -0.500  56.066
#tick.ntp.infoma .GPS.            1 u   45   64    5  180.428  -14.619  21.449
+203.107.6.88    10.137.38.86     2 u   44   64   17   17.410   -1.532  46.437
+dns2.synet.edu. .PTP.            1 u   41   64   17   30.308   -1.038  48.777
+dns1.synet.edu. 202.118.1.47     2 u   47   64   17   22.814   -2.332  48.447
+111.230.189.174 100.122.36.4     2 u   46   64   17   46.926   -1.060  50.993
+ntp5.flashdance 192.36.143.150   2 u  118   64    6  226.584   -6.003  28.797
+139.199.215.251 100.122.36.4     2 u   40   64   17   55.004   -7.861  51.979
+tock.ntp.infoma .GPS.            1 u   48   64   17  217.580  -11.264  48.096
#sv1.ggsrv.de    205.46.178.169   2 u   46   64   17  224.854  -29.628  43.490
+electrode.felix 56.1.129.236     3 u   48   64   17  238.256   22.009  46.528
+120.25.115.20   10.137.53.7      2 u   45   64   17   55.026   -7.769  53.914
+time.cloudflare 10.12.3.34       3 u  111   64    6  181.559   11.466  15.204
```



### exiting nameservercannot be used

为ntpserver 增加hostname
### 时区
`#date -R`
### no server suitabl for synchronization found
`chenge to ip not use hostname`
### on server 
`watch ntpq -p`

