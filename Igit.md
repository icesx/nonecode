git
==================
##git client
### clone
$git clone http://xxx
### commit
$git commit .
$git commit -a .
### push
$git push .
### push delete file
$git add -A
$git commit -m ""
$git push
##git server
+ 安装apache
```
apt-get install apache2 git-core
```
+ 创建仓库
```
mkdir /solar-data/spring-cloud-config.git
cd /solar-data/spring-cloud-config.git
git --bare init
git update-server-info
chown -R www-data:www-data .
```
+ 修改apache配置
```
htpasswd -m -c /solar-data/git-respository/passowrd i
a2enmod dav_fs
rm -rf /etc/apache2/sites-enabled/000-default.conf
#80端口被默认占用了
touch /etc/apache2/conf-enabled/git.conf
----------------------
<VirtualHost *:80>
       
        ServerAdmin webmaster@localhost
        
        DocumentRoot /solar-data/git-respository/

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

<Location /spring-cloud-config.git>
   DAV on
   AuthType Basic
   AuthName "Git"
   AuthUserFile /solar-data/git-respository/password
   Require valid-user
</Location>
</VirtualHost>
----------------------

```

+ 重启apache
```
service apache2 restart
```

##git client

```
git clone http://server/spring-cloud-config.git

cd spring-cloud-config
vi .git/config

#add passowd and use to url as 
	url = http://i:i@solar27/spring-cloud-config.git
#没有修改这句话的时候会出现错误

git push origin master
touch readme.md
git commit -m "init master"
git push origin

error: Cannot access URL http://solar27/spring-cloud-config.git/, return code 22
fatal: git-http-push failed
error: failed to push some refs to 'http://solar27/spring-cloud-config.git'
```
