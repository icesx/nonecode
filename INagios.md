nagios
======

## 基本概念

1. master主机上安装nagios，试了其实不用安装plugin和nrpe
2. slaver上安装nrpe和plugin
3. nagios通过nrpe的check_nrpe命令远程在slaver上执行command
4. command在nrpe/etc/nrpe.cfg中声明，声明command需要plugin的目录（可以在安装的时候，编译选项中指定）
5. nagios/etc/objects/command.cfg中设计远程执行的命令。
6. nagios/etc/servers/xx.cfg中设计需要监控的服务，其中的命令在command.cfg中的指定命令。

## install

#### 环境说明

**Nagios主节点需要安装:**

- nagios
- nagios-plugin
- nrpe

**Nagios从节点需要安装:**

- nagios-plugin
- nrpe

#### 安装Core（主机）

1. 准备

```
sudo apt-get install -y autoconf gcc libc6 make wget unzip apache2 php libapache2-mod-php7.2 libgd-dev
```



2. 安装 core

```
wget https://www.nagios.org/downloads/nagios-core/thanks/
./configure --prefix=/home/bjrdc/software/nagios-4.4.5 --with-nagios-group=bjrdc --with-nagios-user=bjrdc --with-httpd-conf=/etc/apache2/sites-enabled --with-command-group=bjrdc --with-command-user=bjrdc
make all -j8
make install
make install-config
make install-webconf
sudo make install-daemoninit
sudo make install-commandmode
htpasswd -c /home/bjrdc/software/nagios-4.4.5/etc/htpasswd.users nagiosadmin
sudo a2enmod rewrite
sudo a2enmod cgi
sudo service apache2 restart
sudo service nagios restart
```

3. 安装 plugin

   master

```
./configure --prefix=/home/bjrdc/software/nagios-4.4.5 --with-nagios-user=bjrdc --with-nagios-group=bjrdc 
make
make install
```

​		slaver

```
./configure --prefix=/home/bjrdc/software/nagios-plugins-2.3.3 --with-nagios-user=bjrdc --with-nagios-group=bjrdc 
make
make install
```



1. 访问

http://bjrdc51/nagios

#### 安装nper

1. 安装plugin

```
./configure --prefix=/home/bjrdc/software/nagios-plugins-2.3.3 --with-nagios-user=bjrdc --with-nagios-group=bjrdc
```



2. 安装（主机）

```
sudo apt install libssl-dev
./configure --prefix=/home/bjrdc/software/nagios-4.4.5 --with-nrpe-group=bjrdc --with-nrpe-user=bjrdc --with-nagios-user=bjrdc --with-nagios-group=bjrdc
make all
make install-plugin
sudo make install-daemon
make install-config
sudo make install-init
```

3. 安装（被监控机器）

```
./configure --prefix=/home/bjrdc/software/nrpe-3.2.1 --with-nrpe-group=bjrdc --with-nrpe-user=bjrdc --with-nagios-user=bjrdc --with-nagios-group=bjrdc --enable-command-args --with-pluginsdir=/home/bjrdc/software/nagios-plugins-2.3.3/libexec
```

### 配置

1. 配置(被监控机器)

```
vi /home/bjrdc/software/nrpe-3.2.1/etc/nrpe.cfg
change allowed_hosts=127.0.0.1,::1 to allowed_hosts=bjrdc51,::1
dont_blame_nrpe=1
```

```
command[check_users]=/home/bjrdc/software/nagios-plugins-2.3.3/libexec/check_users -w $ARG1$ -c $ARG2$
command[check_load]=/home/bjrdc/software/nagios-plugins-2.3.3/libexec/check_load -w $ARG1$ -c $ARG2$
command[check_disk]=/home/bjrdc/software/nagios-plugins-2.3.3/libexec/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$
command[check_procs]=/home/bjrdc/software/nagios-plugins-2.3.3/libexec/check_procs -w $ARG1$ -c $ARG2$ -s $ARG3$
command[check_procs_args]=/home/bjrdc/software/nagios-plugins-2.3.3/libexec/check_procs  $ARG1$
command[check_swap]=/home/bjrdc/software/nagios-plugins-2.3.3/libexec/check_swap -w $ARG1$ -c $ARG2$

##这里的arg1 对应的就是 xxx.cfg中的command调用的部分，如下
##check_command  check_nrpe_args!check_procs_args!" -c 1:1 -C java -a HMaster"
##这里的check_nrpe_args 对应与 command.cfg 中的对于check_nrpe_args的声明
command[check_swap]=/home/bjrdc/software/nagios-plugins-2.3.3/libexec/check_swap -w $ARG1$ -c $ARG2$
```



2. 配置（主机）
   
   1. nagios.cfg
   
   ```
   cfg_dir=/home/bjrdc/software/nagios-4.4.5/etc/servers
   ```
   
   
   
   1. command.cfg

```
define command{
       command_name    check_nrpe
       command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -t 30 -c $ARG1$
       }
define command{
       command_name    check_nrpe_args
       command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -t 30 -c $ARG1$ -a $ARG2$
       }
```
  2.  group.cfg

```
touch /home/bjrdc/software/nrpe-3.2.1/etc/servers/group.cfg
```
```
define hostgroup{
   hostgroup_name      bjrdc
   alias               bjrdc
   members             localhost,bjrdc100
}
```
3. servicegroup.cfg
```
touch /home/bjrdc/software/nrpe-3.2.1/etc/objects/servicegroups.cfg
```



```
define servicegroup {
    servicegroup_name  hadoop
    alias              Hadoop Service
}
define servicegroup {
    servicegroup_name  mysql
    alias              Mysql Service
}
```



3. host.cfg 
```
touch /home/bjrdc/software/nrpe-3.2.1/etc/servers/host.cfg
```
```
define host{
       use                          linux-server
       host_name                    bjrdc10
       alias                        bjrdc10
       address                      172.16.15.10
       }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Host Alive
       check_command                   check-host-alive
       }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Users
       check_command                   check_nrpe_args!check_users!5 10
       }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             CPU
       check_command                   check_nrpe_args!check_load!15,10,5 30,25,20
       }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Disk Root
       check_command                   check_nrpe_args!check_disk!20% 10% /
       }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Disk /logs
       check_command                   check_nrpe_args!check_disk!20% 10% /logs
       }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Disk /cloud
       check_command                   check_nrpe_args!check_disk!20% 10% /cloud
       }
define service{
      use                             local-service
      host_name                       bjrdc10
      service_description             Procs Zombie
      check_command                   check_nrpe_args!check_procs!5 10 Z
      }
define service{
      use                             local-service
      host_name                       bjrdc10
      service_description             Procs Total
      check_command                   check_nrpe_args!check_procs_args!"-w400 -c600"
      }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Swap Usage
       check_command                   check_nrpe_args!check_swap!20% 10%
       }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Hbase: HMaster
       check_command                   check_nrpe_args!check_procs_args!"-c 1:1 -w 1:1 -C java -a  org.apache.hadoop.hbase.master.HMaster"
        servicegroups                   hadoop
       }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Hadoop: JobHistoryServer
       check_command                   check_nrpe_args!check_procs_args!"-c 1:1 -w 1:1 -C java -a  JobHistoryServer"
       servicegroups                    hadoop
        }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Hadoop: ResourceManager
       check_command                   check_nrpe_args!check_procs_args!"-c 1:1 -w 1:1 -C java -a  ResourceManager"
        servicegroups                   hadoop       
        }
define service{
       use                             local-service
       host_name                       bjrdc10
       service_description             Hadoop: NameNode
       check_command                   check_nrpe_args!check_procs_args!"-c 1:1 -w 1:1 -C java -a  NameNode"
       servicegroups                    hadoop
        }
define service {
    use                     local-service         
    host_name               bjrdc10
    service_description     HBase:Master_HTTP
    check_command           check_http!bjrdc10 -p 16010 -u /master-status
    notifications_enabled   1
        servicegroups                   hadoop
}
```
4. bigdata.cfg
```
touch /home/bjrdc/software/nrpe-3.2.1/etc/servers/bigdata.cfg      
```
```
define service{
       use                             local-service
       host_name                       duangr-2
       service_description             PS: NodeManager
       check_command                   check_nrpe_args!check_procs_args!" -c 1:1 -C java -a HMaster"
       }
define service {
    use                     local-service
    host_name               bjrdc10
    service_description     HBase:Master_HTTP
    check_command           check_http!bjrdc10 -p 16010 -u /master-status
    notifications_enabled   1
    servicegroups           hadoop
}
```
4. 测试
   1. master

```
./check_nrpe  -H bjrdc10 -c check_users -a 5 10
```

​          2. slaver

```
 ~/software/nagios-plugins-2.3.3/libexec/check_procs -w 1:1 -c 1:1 -C java -a QuorumPeerMain
 
```



### 邮件通知

1. Prepare

```
sudo apt install postfix mailutils
echo "test message" | mail -s 'nagios notification' 13824365716@139.com -r 'nagios@bjrdc51.xjgz.com'
#如果收到邮件说明邮件配置完成
```

2. 配置command

修改command.cfg文件，按照如上的命令格式，以便139邮箱能够收到邮件。

```
define command {
    command_name    notify-host-by-email
    command_line    /usr/bin/printf "%b" "Nagios \n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE $\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /usr/bin/mail -s "$NOTIFICATIONTYPE$ Host Alert
: $HOSTNAME$ is $HOSTSTATE$ " $CONTACTEMAIL$ -r nagios@bjrdc.xjgz.com
}

define command {
    command_name    notify-service-by-email
    command_line    /usr/bin/printf "%b" "Nagios \n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /usr/bin/mail -s "$NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ " $CONTACTEMAIL$ -r nagios@bj
rdc.xjgz.com
}

```

3. 修改contact

修改contact.cfg 文件

```
define contact {
    contact_name            nagiosadmin             ; Short name of user
    use                     generic-contact         ; Inherit default values from generic-contact template (defined above)
    alias                   Nagios Admin            ; Full name of user
    email                   13824365716@139.com ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
}
```

## 邮件通知

### 基本概念

ubuntu 中mail命令使用的是mailutils程序，通过postfix来进行邮件的发送。发送邮件需要安装postfix程序，通过postfix可以以本地为邮件服务【postfix】也可以使用smtp。

在阿里云上，由于关闭了25端口，故无法通过本地发送邮件出去，需要使用postfix链接到其他的类似163邮箱的ssl端口465上方可

### 安装

```
sudo apt install postfix mailutils
```



### 配置

#### smtp密码配置

```
echo [smtp.163.com]:465 username:password > /etc/postfix/sasl_password
postmap /etc/postfix/sasl_passwd
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
sudo chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```



#### postfix 配置

`vi /etc/postfix/mail.cf `

```
relayhost = [smtp.163.com]:465
# enable SASL authentication
smtp_sasl_auth_enable = yes
# disallow methods that allow anonymous authentication.
smtp_sasl_security_options = noanonymous
# where to find sasl_passwd
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
# Enable STARTTLS encryption
smtp_use_tls = yes
# !!!IMPORTANT
smtp_tls_wrappermode = yes
smtp_tls_security_level = encrypt
debug_peer_level = 4
smtputf8_enable=no
```

`sudo service postfix restart`

`tail -f /var/log/syslog`

#### 发送邮件

```
echo "This is really cool!" | mailx -s "我正在使用postfix给自己发送邮件" -a "From: 13824365716@163.com" 13824365716@139.com
```

`-a "From:13824365716@163.com"`是发件箱

## 问题处理

1. NRPE: Command 'check_swap!20%!10%' not defined 

   此问题可能是在被监控主机上的nrpe/etc/nrpe.cfg中的check_swap command没有设置

2. 注意：在xxx.cfg 中的配置，如果service_description 不修改，重启nagios，不会让nagios重新加载该配置文件的修改。



