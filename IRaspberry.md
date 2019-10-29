###安装
sudo dd if=/TOOLS/TOOLS/Linux/ISO/2018-04-18-raspbian-stretch-lite.img  of=/dev/mmcblk0 bs=4M
### first
user:pi
passwd:raspberry
### 串口
树莓派自带一个串口/dev/ttyAMA0,但是默认这个串口被getty使用了，所以在开机启动中，注销掉后，才能在py中使用这个串口
```
vi /etc/inittab
#T0:23:respawn:/sbin/getty -L ttyAMA0 115200 vt100
```
###setup
```
sudo raspi-config
```
###wifi
```
sudo ifconfig wlan0 up
sudo iwlist wlan0 scan
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
country=CN
network={
    ssid="testing"
    psk="testingPassword"
}
```
注意：目前测试网口和wifi无法同时用，网口用了后，wifi获取不到ip地址或者设备起不来
### ip
```
auto eth0
iface eth0 inet static
   address 10.253.0.50
   netmask 255.255.255.0
   network 10.253.0.0
   gateway 10.253.0.1
   dns-nameservers 8.8.8.8
```
###keybord
$sudo nano /etc/default/keyboard 

and hit enter. locate the following line

XKBLAYOUT=”gb”

Change the gb to us (This assumes you want a us mapping, if not replace the gb with the two letter code for your country)i

###GPIO
第一列是wiringPi API中的缺省编号，wiringPiSetup()采用这列编号
第二列（Name）往往是转接板的编号
第三列是树莓派板子上的自然编号（左边引脚为1-15，右边引脚为2-26），RPi.GPIO.setmode(GPIO.BOARD)采用这列编号
树莓派主芯片提供商Broadcom的编号方法，相当于调用了WiringPiSetupGpio()或RPi.GPIO.setmode(GPIO.BCM)采用这列编号
wiringPi Pin	Name	Board Pin	BCM GPIO
0	GPIO 0	11	17
1	GPIO 1	12	18
2	GPIO 2	13	21
3	GPIO 3	15	22
4	GPIO 4	16	23
5	GPIO 5	18	24
6	GPIO 6	22	25
7	GPIO 7	7	4
8	SDA	3	0
9	SCL	5	1
10	CE0	24	8
11	CE1	26	7
12	MOSI	19	10
13	MISO	21	9
14	SCLK	23	11
15	TXD	8	14
16	RXD	10	15
Rev.2 新增的引脚：

wiringPi Pin	Name	Board Pin	BCM GPIO
17	GPIO 8		28
18	GPIO 9		29
19	GPIO10		30
20	GPIO11		31
###apt
###ssh
如果出现无法远程的话
sudo rm /etc/ssh/ssh_host_* && sudo dpkg-reconfigure openssh-server
###ssh