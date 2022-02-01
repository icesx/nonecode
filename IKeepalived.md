keepalived
===
## install

### Configuration File for keepalived

```
global_defs {
   notification_email {
     #acassen@firewall.loc
     #failover@firewall.loc
     #sysadmin@firewall.loc
     12157724@qq.com
     13824365716@139.com
     18910826870@189.com
   }
   notification_email_from 12157724@qq.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_script chk_nginx {
     script "/xjgzbj/shell/chk_nginx.keepalived.sh"
     interval 2
     weight 2
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 200
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass qazwsxedc
    }
    virtual_ipaddress {
        61.128.111.179
    }
    track_script {
       chk_nginx 
    }
}

}
```


## mail

keepalived提供了两个hook来支持扩展。
1.keepalived状态发生变化的时候，包括keepalive stop、start、master、salver等。
2.通过checkscript来实现对nginx的监控

在如上两个hook中均可以通过sendmail来发送邮件出去，可以发送到qq邮箱，也可以发送到139邮箱——139邮箱有短信通知功能，故这点是非常的强大的。

通过sendmail来发送邮件很简单，首先linux中需要安装sendmail，注：如果sendmail无法启动，要看看是否系统已经默认安装了postfix，如果安装了，卸载掉它。

sendmail服务启动后，可通过mail命令来发送邮件，命令行使用的话，可以如下
echo "$content"| mail -s "`date`:`hostname`:keepalived changed" $i >/dev/null &
注： >/dev/null &可以非阻塞的方式发送。

如果启用了iptables，需要在iptables中增加规则
 cat /etc/sysconfig/iptables

```
-A INPUT -i eth0 -p tcp --dport 25 -m state --state NEW,ESTABLISHED -j ACCEPT
-A OUTPUT -o eth0 -p tcp --sport 25 -m state --state ESTABLISHED -j ACCEPT
```


如果无法发送的话，查看/var/log/maillog。有可能是域名的问题，很多邮箱已经屏蔽了localhost.localdomain域名，故需要修改系统的域名

注：修改域名需要修改/etc/sysconfig/network文件

一定要记得service sendmail restart

