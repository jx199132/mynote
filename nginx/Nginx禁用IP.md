公司机器配置了Nginx正向代理，用作爬虫使用。

结果很多外网扫描到了该机器，也用作代理服务器，导致网络资源消耗很多。

解决方式：设置Nginx ip访问即可




```
    server{
		allow 192.168.18.0/24;
		allow 111.47.16.194;
		deny all;
		resolver 8.8.8.8;
		listen 8118;
		location / {
			root html;
			index index.html index.htm;
			proxy_pass $scheme://$host$request_uri;
			proxy_set_header HOST $http_host;
			proxy_buffers 256 4k;
			proxy_max_temp_file_size 0k;
			proxy_connect_timeout 30;
			proxy_send_timeout 60;
			proxy_read_timeout 60;
			proxy_next_upstream error timeout invalid_header http_502;
		}
		error_page 500 502 503 504 /50x.html;
		location = /50x.html {
			root html;
		}
	}
```


allow 192.168.18.0/24;
allow 111.47.16.194;
deny all;

允许前面两个ip访问，其他ip禁用