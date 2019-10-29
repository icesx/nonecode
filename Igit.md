git
==================
#git client
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
#git server
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
###保持密码
git config --global credential.helper cache
##branch
###比较本地和远程
git diff master origin/master
### 中文文件名
git config --global core.quotepath false


###多人写作
https://www.liaoxuefeng.com/wiki/896043488029600/900375748016320
```
因此，多人协作的工作模式通常是这样：

首先，可以试图用git push origin <branch-name>推送自己的修改；

如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

如果合并有冲突，则解决冲突，并在本地提交；

没有冲突或者解决掉冲突后，再用git push origin <branch-name>推送就能成功！

如果git pull提示no tracking information，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream-to <branch-name> origin/<branch-name>。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。
```