### 安装

```
sudo apt-get install libpcre3 libpcre3-dev zlib1g-dev gcc make
```



```
./configure --prefix=/home/docker/software/nginx-1.10.0-proxy --with-pcre --with-http_sub_module
```
### 查看模块
```
./sbin/nginx -V
```
### sub_filter
```	
	sub_filter '52.4.1.145:8090' '58.42.241.252:8000';#将页面上的所有的52.4.1.145:8090 替换为 58.42.241.252:8000
	sub_filter '52.4.1.145:8089' '58.42.241.252:8000';
	sub_filter '52.4.1.188:6080' '58.42.241.252:6080';
	sub_filter '52.4.1.188:8580' '58.42.241.252:8580';
	sub_filter '52.4.1.145:18567' '58.42.241.252:8000';
	sub_filter_once off;
	sub_filter_types application/json application/javascript text/javascript;#针对这三总mime，这些mime需要通过抓包或者浏览器的调试工具获取
```
### proxy
```
location / {
		#http://52.4.1.176:8980/;
	        	root html;
			index  index.html index.htm;
	    }
	location /cas{
		proxy_pass http://52.4.1.176:8980/cas;
	}
	location /cdc-manager{
		proxy_pass http://52.4.1.145:18567/cdc-manager;
	}
	location /cdc-search-app{
		proxy_pass http://52.4.1.145:8089/cdc-search-app;
	}
	location /upload{
		proxy_pass http://52.4.1.145:8090/upload;
	}
	location /cloudatacenter{
		proxy_pass http://52.4.1.145:8090/cloudatacenter;
	}
	location /cdc-place-analysis{
		proxy_pass http://52.4.1.145:8089/cdc-place-analysis;
	}
	location /cdc-pre-alarm{
		proxy_pass http://52.4.1.145:8089/cdc-pre-alarm;
	}
	location /cdc-http-api{
		proxy_pass http://52.4.1.145:8089/cdc-http-api;
	}
	location /cdc-map-api{
		proxy_pass http://52.4.1.145:8089/cdc-map-api;
	}
	location /cdc-charts-api{
		proxy_pass http://52.4.1.145:8089/cdc-charts-api;
	}
	    #error_page  404              /404.html;
	location /scitydatacenter{
		proxy_pass http://52.4.1.176:8080/scitydatacenter;
	}
	location /cdc-subscribe{
		proxy_pass http://52.4.1.145:8089/cdc-subscribe;
	}
```
### mp4
```
--with-http_mp4_module
```
```
location /video/ {
mp4;
mp4_buffer_size
1m;
mp4_max_buffer_size 5m;
}
```

#### mp4
Syntax:
mp4
Default:
Context:
location
Reference: mp4
Enables the mp4 streaming feature.
```
location /video {
mp4 on;
}
```
#### mp4_buffer_size
mp4_buffer_size size
Default:512K
	http
Context:
server
location
Reference: mp4_buffer_size
Syntax:
Default:
Sets the buffer size used for processing mp4 file.

#### mp4_max_buffer_size
mp4_max_buffer_size size
10M
http
Context:
server
location
Reference: mp4_max_buffer_size
Syntax:
Default:
Sets the maxium buffer size used for processing mp4 file. If the meta data exceeds thissize Nginx will return a 500 status code and log an error resembling the following:
"/video/file.mp4" mp4 moov atom is too large:
12583268, you may want to increase mp4_max_buffer_size

## 日志切割

```
vi logcron.sh
log_dir="/logs/nginx"
pid="/logs/nginx/nginx.pid"
date_dir=`date +%Y_%m_%d_%H`
/bin/mv ${log_dir}/access.log ${log_dir}/${date_dir}_access.log
kill -USR1 `cat ${pid}`
```

## 防流量攻击
>nginx可以通过ngx_http_limit_conn_module和ngx_http_limit_req_module配置来限制ip在同一时间段的访问次数
>
>**ngx_http_limit_conn_module**：该模块用于限制每个定义的密钥的连接数，特别是单个IP地址的连接数．使用limit_conn_zone和limit_conn指令
>
>**ngx_http_limit_req_module**：用于限制每一个定义的密钥的请求的处理速率，特别是从一个单一的IP地址的请求的处理速率。使用“泄漏桶”方法进行限制．指令：limit_req_zone和limit_req．

```
# 限制单个IP的连接数示例
http { 
  limit_conn_zone $binary_remote_addr zone=addr：10m; 
　　 #定义一个名为addr的limit_req_zone用来存储session，大小是10M内存，
　　 ...
　　  server { 
    	limit_conn addr 10; 　　#连接数限制
      
```

```
#限制来自单个IP地址的请求的处理速率
http {
  limit_req_zone $binary_remote_addr zone=perip:10m rate=10r/s;
  ...
  server {
    ...
      limit_req zone=perip burst=5 nodelay;　　#漏桶数为５个．也就是队列数．nodelay:不启用延迟．
      limit_req zone=perserver burst=10;　　　　#限制nginx的处理速率为每秒10个
    }
```

