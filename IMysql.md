mysql
=========

### 安装
```
$tar –zxvf mysql-enterprise-5.0.30-linux-i686-glibc23.tar.gz –C /usr/local/
$cd /usr/local/
$ln –s /cloud/mysql-enterprise-5.0.30-linux-i686-glibc23 mysql
$cd mysql
$groupadd mysql
$useradd -g mysql mysql
$chown -R mysql：mysql .
$sudo usermod  -a -G mysql docker [当前docker用户加入mysql组]
$sudo scripts/mysql_install_db --user=mysql
$cp /usr/local/mysql/support-files/my-medium.cnf /etc/my.cnf
$sudo ./mysqld_safe --user=mysql& 
```
### 配置
```
$sudo cp my-default.cnf /etc/my.cnf
$vi /etc/my.cnf
```
```
[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci


[client]
default-character-set=utf8


```

### 创建用户
```
GRANT  ALTER,USAGE,DROP,SELECT, INSERT, UPDATE, DELETE, CREATE,INDEX,SHOW VIEW ,CREATE TEMPORARY TABLES,EXECUTE ON cdc.* TO 'docker'@'%' IDENTIFIED BY  'xjgz@123';

GRANT  ALTER,USAGE,DROP,SELECT, INSERT, UPDATE, DELETE, CREATE,INDEX,SHOW VIEW ,CREATE TEMPORARY TABLES,EXECUTE ON bdp.* TO 'bjrdc'@'%' IDENTIFIED BY  'xjgz@123';
```
### 权限处理
```
./mysqld_safe --skip-grant-table
```
### 异常处理

ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

to do this
```
$ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock
```
### 关闭
```
$mysqladmin -u root -p shutdown
```

### 修改密码
```
$mysql -u root -p
USE mysql
update user set host='%', Password=password('ipanelappsmgs123456') where User='root'£»
flush privileges;
```
### 显示链接
```
$./mysqladmin -uadmin -p -h10.140.1.1 processlist
$./mysqladmin  -uadmin -p -h10.140.1.1 status
```
### 异常处理
Permissions issue can't find file /mysql/host.frm error code errno 13£º
```
chown -R root . 
chown -R mysql data
chgrp -R mysql . 
```
### 参数调整
```
log-queries-not-using-indexes = nouseindex.log 
log-error=log-error.log
log=log-query.log
log-queries-not-using-indexes
log-warnings=2
log-slow-queries=log-slow-query.log
log-update=log-update.log
long_query_time=2

general_log=ON
general_log_file=/tmp/mysql.log
```
### 删除多余链接
```
for i in `mysql -u root -pgehua -e "show processlist;"|grep Sleep|awk '{print $1}'`; do mysql -uroot -pgehua -e "kill $i;";done; 
```
### 动态调整参数
```
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
```
### 自动备份

#### imysql_backup[mine].sh
```
#!/bin/bash
#mql_backup.sh: backup mysql databases and keep newest 5 days backup.
#
# ----------------------------------------------------------------------
# This is a free shell script under GNU GPL version 2.0 or above
# Copyright (C) 2006 Sam Tang
# Feedback/comment/suggestions : http://www.real-blog.com/
# ----------------------------------------------------------------------
# your mysql login information
# db_user is mysql username
# db_passwd is mysql password
# db_host is mysql host
# -----------------------------
db_user="root"
db_passwd="i"
db_host="localhost"

# the directory for story your backup file.
backup_dir="/i/mysql_auto_backup"
# date format for backup file (dd-mm-yyyy)
time="$(date +"%Y_%m_%d")"
# mysql, mysqldump and some other bin's path
MYSQL="/usr/local/mysql/bin/mysql"
MYSQLDUMP="/usr/local/mysql/bin/mysqldump"
MKDIR="/bin/mkdir"
RM="/bin/rm"
MV="/bin/mv"
GZIP="/bin/gzip"
# check the directory for store backup is writeable
test ! -w $backup_dir && echo "Error: $backup_dir is un-writeable." && exit 0
# the directory for story the newest backup
test ! -d "$backup_dir/backup.0/" && $MKDIR "$backup_dir/backup.0/"
# get all databases
all_db="$($MYSQL -u $db_user -h $db_host -p$db_passwd -Bse 'show databases')"
for db in $all_db
do
	$MYSQLDUMP -u $db_user -h $db_host -p$db_passwd $db | $GZIP -9 > "$backup_dir/backup.0/$time.$db.gz"
done	
	# delete the oldest backup
	test -d "$backup_dir/backup.16/" && $RM -rf "$backup_dir/backup.16"
	# rotate backup directory
for int in 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0
do
	if(test -d "$backup_dir"/backup."$int")
	then
		next_int=`expr $int + 1`
		$MV "$backup_dir"/backup."$int" "$backup_dir"/backup."$next_int"
	fi
done
exit 0;
```
### 查询链接
`./mysqladmin -uadmin -p -h10.140.1.1 processlist`

### 查看链接状态
`./mysqladmin  -uadmin -p -h10.140.1.1 status`
