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



## DVWA

[官方网](https://github.com/digininja/DVWA)

### 安装php

### 下载安装包

### 安装配置

### Database Setup

To set up the database, simply click on the `Setup DVWA` button in the main menu, then click on the `Create / Reset Database` button. This will create / reset the database for you with some data in.

If you receive an error while trying to create your database, make sure your database credentials are correct within `./config/config.inc.php`. *This differs from config.inc.php.dist, which is an example file.*

The variables are set to the following by default:

```
$_DVWA[ 'db_user' ] = 'dvwa';
$_DVWA[ 'db_password' ] = 'p@ssw0rd';
$_DVWA[ 'db_database' ] = 'dvwa';
```

Note, if you are using MariaDB rather than MySQL (MariaDB is default in Kali), then you can't use the database root user, you must create a new database user. To do this, connect to the database as the root user then use the following commands:

```
mysql> create database dvwa;
Query OK, 1 row affected (0.00 sec)

mysql> create user dvwa@localhost identified by 'p@ssw0rd';
Query OK, 0 rows affected (0.01 sec)

mysql> grant all on dvwa.* to dvwa@localhost;
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

### Other Configuration

Depending on your Operating System, as well as version of PHP, you may wish to alter the default configuration. The location of the files will be different on a per-machine basis.

**Folder Permissions**:

- `./hackable/uploads/` - Needs to be writeable by the web service (for File Upload).
- `./external/phpids/0.6/lib/IDS/tmp/phpids_log.txt` - Needs to be writable by the web service (if you wish to use PHPIDS).

**PHP configuration**:

- `allow_url_include = on` - Allows for Remote File Inclusions (RFI) [[allow_url_include](https://secure.php.net/manual/en/filesystem.configuration.php#ini.allow-url-include)]
- `allow_url_fopen = on` - Allows for Remote File Inclusions (RFI) [[allow_url_fopen](https://secure.php.net/manual/en/filesystem.configuration.php#ini.allow-url-fopen)]
- `safe_mode = off` - (If PHP <= v5.4) Allows for SQL Injection (SQLi) [[safe_mode](https://secure.php.net/manual/en/features.safe-mode.php)]
- `magic_quotes_gpc = off` - (If PHP <= v5.4) Allows for SQL Injection (SQLi) [[magic_quotes_gpc](https://secure.php.net/manual/en/security.magicquotes.php)]
- `display_errors = off` - (Optional) Hides PHP warning messages to make it less verbose [[display_errors](https://secure.php.net/manual/en/errorfunc.configuration.php#ini.display-errors)]

**File: `config/config.inc.php`**:

- `$_DVWA[ 'recaptcha_public_key' ]` & `$_DVWA[ 'recaptcha_private_key' ]` - These values need to be generated from: https://www.google.com/recaptcha/admin/create

### Default Credentials

**Default username = `admin`**

**Default password = `password`**

*...can easily be brute forced ;)*

Login URL: http://127.0.0.1/login.php

*Note: This will be different if you installed DVWA into a different directory.*



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

