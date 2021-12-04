Izentao
======

## php7 install

```sh
sudo apt install libapr1-dev libaprutil1-dev  libpcre3-dev pkg-config libxml2-dev libssl-dev libonig-dev libcurl-openssl1.0-dev
```



```sh
./configure --with-apxs2=/home/bjrdc/software/httpd/bin/apxs --with-zlib --prefix=/home/bjrdc/software/php --without-sqlite3 --enable-mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --without-pdo-sqlite --prefix=/home/bjrdc/software/php --with-openssl --enable-mbstring --with-curl
```

php.ini

```sh
cp /home/bjrdc/tools/php7.4.7/php.ini-production /home/bjrdc/software/php/lib/php.ini
```



### apache install

```sh
./configure --prefix=/home/bjrdc/software/httpd
```

httpd/conf/httpd.conf
增加php的filesmatch

  ```
<FilesMatch \.php$>
SetHandler application/x-httpd-php
</FilesMatch>
  ```

## mysql



## zentao install

