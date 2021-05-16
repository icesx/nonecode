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
### Server

> Change the Nginx server name in source file(src/http/ngx_http_header_filter_module.c) to " My-Server". After that, compiled the nginx. But its not working when I load the url. Strange here is I can see my updated Signature when I use curl command. But this same is not updated in browser.

### sub_filter

```	nginx
location / {
    proxy_set_header Accept-Encoding "";#防止压缩引起的无法过滤
	sub_filter '52.4.1.145:8090' '58.42.241.252:8000';#将页面上的所有的52.4.1.145:8090 替换为 58.42.241.252:8000
	sub_filter '52.4.1.145:8089' '58.42.241.252:8000';
	sub_filter '52.4.1.188:6080' '58.42.241.252:6080';
	sub_filter '52.4.1.188:8580' '58.42.241.252:8580';
	sub_filter '52.4.1.145:18567' '58.42.241.252:8000';
	sub_filter_once off;
	sub_filter_types application/json application/javascript text/javascript;#针对这三总mime，这些mime需要通过抓包或者浏览器的调试工具获取
}
```
### proxy
```nginx
location / {
		#http://52.4.1.176:8980/;
	        	root html;
			index  index.html index.htm;
	    }
	location /cas/{
		proxy_pass http://52.4.1.176:8980/cas/;
	}
	location /cdc-manager{
		proxy_pass http://52.4.1.145:18567;
	}
```
### auth

```
sudo apt install apache2-utils
cd $nginx/config
htpasswd password user
```

配置nginx.conf

```nginx
http {
...
    auth_basic "password";
    auth_basic_user_file password;
...    
```



### mp4

```
--with-http_mp4_module
```
```nginx
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
```nginx
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

```sh
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

```nginx
# 限制单个IP的连接数示例
http { 
  limit_conn_zone $binary_remote_addr zone=addr：10m; 
　　 #定义一个名为addr的limit_req_zone用来存储session，大小是10M内存，
　　 ...
　　  server { 
    	limit_conn addr 10; 　　#连接数限制
      
```

```nginx
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

## location 匹配

http://nginx.org/en/docs/http/ngx_http_core_module.html#location

### 基本语法

```
Syntax:	location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
Default:	—
Context:	server, location
```

Let’s illustrate the above by an example:

> ```nginx
> location = / {
>     [ configuration A ]
> }
> 
> location / {
>     [ configuration B ]
> }
> 
> location /documents/ {
>     [ configuration C ]
> }
> 
> location ^~ /images/ {
>     [ configuration D ]
> }
> 
> location ~* \.(gif|jpg|jpeg)$ {
>     [ configuration E ]
> }
> ```

The “`/`” request will match configuration A, 

the “`/index.html`” request will match configuration B, 

the “`/documents/document.html`” request will match configuration C,

 the “`/images/1.gif`” request will match configuration D, and the “`/documents/1.jpg`” request will match configuration E.



### 符号说明

| ~    | 正则匹配，区分大小写                                         |
| ---- | ------------------------------------------------------------ |
| ~*   | 正则匹配，不区分大小写                                       |
| ^~   | 普通字符匹配，如果该选项匹配，则，只匹配改选项，不再向下匹配其他选项 |
| =    | 普通字符匹配，精确匹配                                       |
| @    | 定义一个命名的 location，用于内部定向，例如 error_page，try_files |

### 示例

1. 匹配/

   ```
   location = /
   ```

2. 匹配js下的所有js文件

   ```
   location ~* /js/.*/.js
   ```

3. 精确匹配 / ，主机名后面不能带任何字符串

   ```
   location  /
   ```

4. 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索

   ```
   location /documents/
   ```

5. 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条

   ```
   location ^~ /images/
   ```

6. 匹配所有以 gif,jpg或jpeg 结尾的请求

   ```
   location ~* .(gif|jpg|jpeg)$
   ```

## cache

### 简单配置

```nginx
http {
    proxy_cache_path  /data/nginx/cache  levels=1:2    keys_zone=STATIC:10m
    inactive=24h  max_size=1g;
    server {
        location / {
            proxy_pass             http://1.2.3.4;
            proxy_set_header       Host $host;
            proxy_buffering        on;
            proxy_cache            STATIC;
            proxy_cache_valid      200  1d;
            proxy_cache_use_stale  error timeout invalid_header updating
                                   http_500 http_502 http_503 http_504;
        }
    }
}
```
 自己测试成功的配置

```nginx
location /moa-static-demo/ {
	proxy_pass  http://bjrdc82:10800/;
	proxy_cache image_cache;
	add_header X-Cache-Status $upstream_cache_status;
	proxy_cache_valid      200  1d;
}
```

### timeout

```nginx
http{
    send_timeout 15s;
    proxy_connect_timeout 60s;
    proxy_read_timeout 60s;
}
```

### log_format

```nginx
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" '
                      '"$request_time " "$upstream_response_time" ';
    access_log  logs/access.log  main;
}
```



### 针对图片缓存（包含反向代理）

```nginx
    upstream backserver {
      ip_hash;
      server bjrdc251:8080 weight=10;
      server bjrdc252:8080 weight=2;
    }
    proxy_cache_path  /home/bjrdc/cache/nginx  levels=1:2    keys_zone=image_cache:10m
    inactive=24h  max_size=1g;
    server {
        listen       9080;
        server_name  localhost;
    location ~* /back/.*\.(jpg|jpeg|gif|png)$ {
        proxy_pass  http://backserver;
        proxy_cache image_cache;
        add_header X-Cache-Status $upstream_cache_status;
        proxy_cache_valid      200  1d;
    }
    location /back {
        proxy_pass http://backserver;
    }
```

> 如果通过正则表达式匹配地址的时候，`proxy_pass` 后的路径不能待url和/

## 负载均衡

### 基本配置

```nginx
cat ../conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
upstream backserver { 
    hash $request_uri; 
    server bjrdc251:8080 weight=10; 
    server bjrdc252:8080 weight=2; 
}
    server {
        listen       9080;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
    location /back/ {
        proxy_pass http://backserver/back/;
        add_header Access-Control-Allow-Origin *;
    }
        location / {
            root   html;
            index  index.html index.htm;
        }

     
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```

代理机器上如下配置

```nginx
    server {
        listen       8080;
        }
```



不需要作其他配置自动实现负载均衡和健康检测

```
 for i in {1..10000}; do curl bjrdc250:9080/back/index.html; done
```

