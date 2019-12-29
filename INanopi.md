nanopi
=============

### install

Insert this card into your board's BOOT slot and power on (with a 5V/2A power source). If the PWR LED is on and the STAT LED is blinking this indicates your board has successfully booted.



```
cu -l ttyAMA0 -s 115200
pi
pi
sudo eflasher

```

```
auto eth0
iface eth0 inet static
address 192.168.31.10
netmask 255.255.255.0
gateway 192.168.31.1
```

```
sudo systemctl enable dnsmasq
sudo vi /etc/dnsmasq.conf
	server=114.114.114.114
sudo systemctl start dnsmasq
```

