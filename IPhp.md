php
==========
## prepare
1. 安装apache
2. 安装组件
```
sudo apt install libxml2-dev libpng-dev
```
## php5
1. 第一种
```
./configure --with-apxs2=/xjgzbj/apache/bin/apxs --with-mysql --prefix=/usr/local/php --with-config-file-path=/xjgz/php/conf --with-mcrypt=/usr/local/libs/libmcrypt --with-curl=/usr/local/libs/curl --with-gd --with-png-dir=/usr/local/libs/libpng --with-pdo-mysql --with-mysql-sock=/tmp/mysql.sock
```
2. with mysql
```
./configure --with-apxs2=/xjgzbj/apache/bin/apxs --with-mysql --prefix=/usr/local/php --with-config-file-path=/xjgz/php/conf --with-pdo-mysql --with-mysql-sock=/tmp/mysql.sock
```

3. with jpeg mysql
```
./configure --with-apxs2=/home/xjgzbj/apache/bin/apxs --with-mysql --prefix=/home/xjgzbj/php --with-config-file-path=/home/xjgzbj/php/conf --with-pdo-mysql --with-mysql-sock=/var/lib/mysql/mysql.sock --with-jpeg-dir=/home/xjgzbj/libs/libjpeg --with-gd --with-png-dir=/home/xjgzbj/libs/libpng --with-freetype-dir=/usr/include/freetype2/freetype
```
5.  with jpeg zip mysql  png
```
./configure --with-apxs2=/TOOLS/apache/bin/apxs --with-mysql --prefix=/TOOLS/php --with-config-file-path=/TOOLS/php/conf --with-curl=/TOOLS/libs/curl --with-gd --with-png-dir=/TOOLS/libs/libpng --with-jpeg-dir=/TOOLS/libs/libjpeg --with-freetype-dir=/TOOLS/libs/freetype2 --with-zlib-dir=/TOOLS/libs/zlib
```
6.  with jpeg zip mysql  png libxml
```
./configure --with-apxs2=/home/xjgzbj/apache/bin/apxs --with-mysql --prefix=/home/xjgzbj/php --with-config-file-path=/home/xjgzbj/php/conf --with-pdo-mysql --with-jpeg-dir=/home/xjgzbj/libs/libjpeg --with-gd --with-png-dir=/home/xjgzbj/libs/libpng --with-freetype-dir=/home/xjgzbj/libs/freetype --with-libxml-dir=/home/xjgzbj/libs/libxml2
```
## php7

0. apache 安装

   首先安装必要的支持包

```
sudo apt install libapr1-dev libaprutil1-dev  libpcre3-dev pkg-config libxml2-dev libssl-dev libonig-dev libcurl-openssl1.0-dev
```

   安装

```
./configure --prefix=/home/bjrdc/httpd-2.4.42
```

### with apache

1. php7 安装

   ```
   ./configure --with-apxs2=/home/bjrdc/software/httpd-2.4.42/bin/apxs --with-zlib --prefix=/home/bjrdc/software/php-7.3.12 --with-mysqli=/usr/local/mysql/bin/mysql_config --without-sqlite3 --enable-mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --without-pdo-sqlite --prefix=/home/bjrdc/software/php
   make -j6
   make install
   libtool --finish /home/bjrdc/tools/php-7.4.7/libs
   ```

   此后，apache的config/httpd.conf文件应该增加了一行（如果没有自动增加的话）

   ```
   LoadModule php7_module        modules/libphp7.so
   ```

2. httpd/conf/httpd.conf
     增加php的filesmatch

  ```
  <FilesMatch \.php$>
  SetHandler application/x-httpd-php
  </FilesMatch>
  ```

3. 配置php.ini

   默认安装的时候，php.ini没有自动安装，需要使用如下命令

   ```
   cp /home/bjrdc/tools/php7.4.7/php.ini-production /home/bjrdc/software/php/lib/
   ```

   重启apache

   ```
   httpd -k restart
   ```

4. 修该mysql参数

   vi php.ini

   ```
   mysql.default_socket=/var/run/mysqld/mysqld.sock
   pdo_mysql.default_socket=/var/run/mysqld/mysqld.sock
   #/var/run/mysqld/mysqld.sock many is not there in your server
   ```


### with nginx

1. 安装php

   ```
   ./configure --with-apxs2=/home/liufeng/software/httpd-2.4.42/bin/apxs --with-zlib --prefix=/home/liufeng/software/php-7.3.12 --without-sqlite3 --enable-mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --without-pdo-sqlite --prefix=/home/liufeng/software/php --with-openssl --enable-fpm
   ```

2. 启动fpm

   ```
   $php_home/sbin/php-fpm
   ```

3. nginx 配置

   ```
           location / {
               root  	path/to/wordpress;
               index  index.html index.htm index.php;
           }
   ```

   

   ```
           location ~ \.php$ {
               root           /home/liufeng/software/wordpress;
               fastcgi_pass 127.0.0.1:9000;
               fastcgi_index  index.php;
               fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
               include        fastcgi_params;
           }
   ```

   

### test

1. 创建phpinfo.php

   ```
   cat > httpd/htdocs/phpinfo.php  <<EOF
   <?php phpinfo(); ?>
   EOF
   ```

2. 访问

   ```
   curl http://bjrdc215:8080/phpinfo.php
   ```

   

### zentao

```
./configure --with-apxs2=/home/bjrdc/software/httpd/bin/apxs --with-zlib --prefix=/home/bjrdc/software/php --without-sqlite3 --enable-mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --without-pdo-sqlite --prefix=/home/bjrdc/software/php --with-openssl --enable-mbstring --with-curl
```

