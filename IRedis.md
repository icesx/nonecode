###安装
	$reidis_home/make MALLOC=libc
###单机
	A、启动
		./redis-server ../redis.conf
	B、命令
		#./redis-cli KEYS "SITE_ALARM_PERSON_PREFIX*"|xargs ./redis-cli DEL
		#./redis-cli KEYS "*"
	C、save
		#redis 127.0.0.1:6379> SAVE 
		会产生如下文件：/home/docker/dump.rdb
		#redis 127.0.0.1:6379> BGSAVE
	D、恢复数据
		如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令，如下所示：
		#redis 127.0.0.1:6379> CONFIG GET dir
		1) "dir"
		2) "/usr/local/redis/bin"
	E、$./redis-cli config get maxclients
	F、$./redis-cli config get "*"
	F、redis :6379 > shutdown
	H. 关闭服务# redis-cli shutdown 
###配置存储路径
	$vi $redis_home/redis.conf
	dir /cloud/redis
###配置日志
	dir /cloud/reids.log
###刷写数据
	$./redis-cli save
###清理数据
	
