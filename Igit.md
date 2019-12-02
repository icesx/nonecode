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
###换行符^M忽略
git config --global core.whitespace cr-at-eol
##branch
###比较本地和远程
git diff master origin/master
### 中文文件名
git config --global core.quotepath false
#git 的正确用法
##branch
远程master分支
本地dev分支
##正常工作流程
1. 从服务器checkout master分支
	git clone http:///.xx.git
	git checkout master
2. 创建本地 dev 分支
	git branch dev
3. 切换到dev分支开发代码
	git checkout dev
4. 提交之前从切换到master分支
	git checkout master
5. 将本地dev代码merge到master
	git merge dev
6. 提交代码
	git push
##异常情况
###merge冲突
如果在从dev merge到master后，提交的时候，发现master remote已经更新，此时可以将master还原，再pull之后在进行merge
	git reset --hard
	如果要reset到一个指定版本，可以使用 git log查看版本后，使用命令
	git reset --hard 436d03844f742bdb077dbfeb9e42c2cb687ae384
	git pull
	git merge dev

###删除Untracked files
	git clean -f
###强制更新
	git fetch --all    //只是下载代码到本地，不进行合并操
	git reset --hard origin/master
	
	
	
	

###同时修改
error: Your local changes to the following files would be overwritten by merge:
Please commit your changes or stash them before you merge.


解决办法：

1、服务器代码合并本地代码
```
$ git stash     //暂存当前正在进行的工作。
$ git pull   origin master //拉取服务器的代码

$ git stash pop //合并暂存的代码
```

