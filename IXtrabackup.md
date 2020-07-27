xtrabackup
======

### install

```
sudo apt install percona-xtrabackup
```

### 使用

1. privileges

   ```
   CREATE USER 'bkupdbuser'@'%' IDENTIFIED BY '123123';
   GRANT RELOAD,LOCK TABLES,REPLICATION CLIENT,PROCESS ON *.* TO 'bkupdbuser'@'%';
   FLUSH PRIVILEGES;
   ```

2. backup fullbackup

   ```
   innobackupex --defaults-file=/usr/local/mysql/my.cnf --host=localhost --socket=/tmp/mysql.sock --user=root backup
   ```

3. 增量备份

   ```
   innobackupex  --user=root --incremental-basedir=/data/backup/2015-09-18_16-35-12 --incremental /data/backup/
   ```

4. 还原

   ```
   
   ```

   

