subversion
===============
### install
	audo apt install subversion
### config
####create
	svnadmin create my-project
####authz
	cd my-project
	vi config/authz
```
[groups]
faced=ct,ly,zwf

[face-entrance:/]
@faced=rw
```
####passwd
	vi config/passwd
```
[users]
ct=chengtao@123
ly=liaoyi@123
zwf=zhongwf@123

```
####server
	vi config/svnserver.conf
```
[general]
auth-access = write
password-db = passwd
authz-db = authz

```
### start
	svnserve --listen-port=31690 -d --root=/svn --log-file=/svn/svn.log

###with apache
```
./configure --enable-dav --enable-so --prefix=/usr/local/apache2/
./configure --prefix=/xjgzbj/subversion --with-apxs=/xjgzbj/apache/bin/apxs --with-apr=/xjgzbj/apache/ --with-apr-util=/xjgzbj/apache/ --with-openssl --with-zlib --enable-maintainer-mode
```
###问题处理
+ svn: E220001: Unreadable path encountered; access denied

	在项目的conf/svnserve.conf 中, 设置 anon-access = none 即可. 然后重启Subversion 服务.