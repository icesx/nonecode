###安装
$./mysqld_safe --user=mysql&

$chown -R mysql mysqld . 


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
