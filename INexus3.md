nexus
==========
##安装
1. 官网下载最新版本，解压缩
	https://www.sonatype.com/download-oss-sonatype
2. 修改端口
	vi ${nexus_home}/etc/nexus-default.properties
##密码修改
1. 停服
2. 进入OrientDB控制台
	cd ${nexus_home}
	java -jar ./lib/support/nexus-orient-console.jar

3. 在控制台执行：
	connect plocal:/home/xiaoban/nexus-repository/nexus3/db/security admin admin

4. 重置密码为admin123

	update user SET password="$shiro1$SHA-512$1024$NE+wqQq/TmjZMvfI7ENh/g==$V4yPw8T64UQ6GfJfxYq2hLsVrBY8D1v+bktfOxGdt4b/9BthpWPNUy/CBk6V9iA0nHpzYzJFWO8v/tZFtES8CA==" UPSERT WHERE id="admin"
5. 启动服务

##
