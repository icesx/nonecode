### 安装
 	./configure --prefix=/home/docker/software/nginx-1.10.0-proxy --with-pcre --with-http_sub_module
### 查看模块
	./sbin/nginx -V
### sub_filter
	sub_filter '52.4.1.145:8090' '58.42.241.252:8000';#将页面上的所有的52.4.1.145:8090 替换为 58.42.241.252:8000
	sub_filter '52.4.1.145:8089' '58.42.241.252:8000';
	sub_filter '52.4.1.188:6080' '58.42.241.252:6080';
	sub_filter '52.4.1.188:8580' '58.42.241.252:8580';
	sub_filter '52.4.1.145:18567' '58.42.241.252:8000';
	sub_filter_once off;
	sub_filter_types application/json application/javascript text/javascript;#针对这三总mime，这些mime需要通过抓包或者浏览器的调试工具获取
### proxy
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

###mp4
--with-http_mp4_module
```
location /video/ {
mp4;
mp4_buffer_size
1m;
mp4_max_buffer_size 5m;
}
```

####mp4
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
####mp4_buffer_size
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
####mp4_max_buffer_size
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