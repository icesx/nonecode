###安装
	$tar –zxvf mysql-enterprise-5.0.30-linux-i686-glibc23.tar.gz –C /usr/local/
	$cd /usr/local/
	$ln –s /cloud/mysql-enterprise-5.0.30-linux-i686-glibc23 mysql
	$cd mysql
	$groupadd mysql
	$useradd -g mysql mysql
	$chown -R mysql：mysql .
	$sudo usermod  -a -G mysql docker [当前docker用户加入mysql组]
	$scripts/mysql_install_db --user=mysql
	$cp /usr/local/mysql/support-files/my-medium.cnf /etc/my.cnf
	$./mysqld_safe --user=mysql& 
###配置
	$sudo cp my-default.cnf /etc/my.cnf
	$vi /etc/my.cnf
```
[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci


[client]
default-character-set=utf8


```

###创建用户
	Grant alter,usage,drop,select,insert,update,delete,create,index,show view,create temporary tables,execute on cdc.*TO'docker'@'%' identified by 'xjgz@123'; 
###权限处理
	./mysqld_safe --skip-grant-table
	
###异常处理

ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

to do this
$ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock

to do this
$mysqladmin -u root -p shutdown


to do this
$mysql -u root -p

USE mysql
update user set host='%', Password=password('ipanelappsmgs123456') where User='root'£»
flush privileges;
###显示链接
$./mysqladmin -uadmin -p -h10.140.1.1 processlist


$./mysqladmin  -uadmin -p -h10.140.1.1 status
###异常处理
Permissions issue can't find file /mysql/host.frm error code errno 13£º
	chown -R root . 
	chown -R mysql data
	chgrp -R mysql . 

###参数调整
log-queries-not-using-indexes = nouseindex.log 
log-error=log-error.log
log=log-query.log
log-queries-not-using-indexes
log-warnings=2
log-slow-queries=log-slow-query.log
log-update=log-update.log
long_query_time=2
 
###删除多余链接
for i in `mysql -u root -pgehua -e "show processlist;"|grep Sleep|awk '{print $1}'`; do mysql -uroot -pgehua -e "kill $i;";done; 
###动态调整参数
mysql> show variables like '%sort_buffer_size%';
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 6291448    |
+---------------------------+------------+
1 rows in set (0.00 sec)


mysql> set SESSION sort_buffer_size=7000000;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like '%sort_buffer_size%';        
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 7000000    |
+---------------------------+------------+
1 rows in set (0.00 sec)


mysql> show variables like '%sort_buffer_size%';        
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 6291448    |
+---------------------------+------------+
1 rows in set (0.00 sec)


mysql> show variables like '%sort_buffer_size%';        
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 6291448    |
+---------------------------+------------+
1 rows in set (0.00 sec)

ÓÃset GLOBAL ÃüÁîÉèÖÃÈ«ŸÖ±äÁ¿

mysql> set GLOBAL sort_buffer_size = 7000000;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like '%sort_buffer_size%';         
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 6291448    |
+---------------------------+------------+
1 rows in set (0.00 sec)

mysql> show variables like '%sort_buffer_size%';
+---------------------------+------------+
| Variable_name             | Value      |
+---------------------------+------------+
| sort_buffer_size          | 7000000    |
+---------------------------+------------+
1 rows in set (0.00 sec)
