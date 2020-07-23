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
#### php7
0. apache 安装
```
./configure --prefix=/home/bjrdc/software/httpd-2.4.14 --disable-apr
make -j6
make install
```
1. php7 安装
```
 ./configure --with-apxs2=/home/bjrdc/software/httpd-2.4.14/bin/apxs --with-zlib --prefix=/home/bjrdc/software/php-7.3.12 --with-gd  --enable-zip --with-mysqli=/usr/local/mysql/bin/mysql_config 
make -j6
make install
```
此后，apache的config/httpd.conf文件应该增加了一行
```
LoadModule php7_module        modules/libphp7.so
```
2. httpd/conf/httpd.conf
增加
```
<FilesMatch \.php$>
SetHandler application/x-httpd-php
</FilesMatch>
```
#### test



## nginx with php

