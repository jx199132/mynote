# 概念
虚拟主机是一种特殊的软硬件技术，它可以将网络上的每一台计算机分成多个虚拟主机，每个虚拟主机可以独立对外提供www服务，这样就可以实现一台主机对外提供多个web服务，每个虚拟主机之间是独立的，互不影响。
# Nginx中虚拟主机的配置
nginx可以实现虚拟主机的配置，nginx支持三种类型的虚拟主机配置。
- 1、基于域名的虚拟主机 （server_name来区分虚拟主机——应用：外部网站）
- 2、基于ip的虚拟主机， （一块主机绑定多个ip地址）
- 3、基于端口的虚拟主机 （端口来区分虚拟主机——应用：公司内部网站，外部网站的管理后台）


## 基于域名的虚拟主机配置

在http块中加入 如下配置：

```
server {
	listen 80;
	server_name www.myhost1.com;
	
	location / {
		root myhost1;
		index index.html;
	}
}
	
server {
	listen 80;
	server_name www.myhost2.com;
	
	location / {
		root myhost2;
		index index.html;
	}
}
```

在本机修改 hosts文件

```
192.168.72.128 www.myhost2.com

192.168.72.128 www.myhost1.com
```

在nginx目录下 /usr/local/nginx/  下分别创建文件夹  myhost1 、 myhost2


```
/usr/local/nginx/myhost1
/usr/local/nginx/myhost2
```


并且分别在两个文件夹中创建index.html文件


```
/usr/local/nginx/myhost1/index.html
/usr/local/nginx/myhost2/index.html
```

重启nginx，并且在浏览器分别访问两个域名

```
/usr/local/nginx/sbin/nginx -s reload
```

http://www.myhost1.com/

http://www.myhost2.com/


## 基于端口的虚拟主机配置

在上面基础上，在Nginx配置文件中添加一个server

```
server {
	listen 86;
	server_name www.myhost1.com;
	
	location / {
		root myhost1;
		index index_86.html;
	}
}
```

在/usr/local/nginx/myhost1/下新建一个index_86.html

重启nginx并且通过浏览器访问

http://www.myhost1.com:86/

http://www.myhost1.com/

## 基于IP的虚拟主机配置

在上面基础上，在Nginx配置文件中添加一个Server块

```
server {
	listen 80;
	server_name 192.168.72.128;
	
	location / {
		root ip;
		index index.html;
	}
}
```

在nginx主目录新建一个文件夹ip，并且新建一个index.html文件

重启nginx

访问浏览器

http://192.168.72.128/


## 附上nginx全部配置文件

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
	
	
	server {
		listen 80;
		server_name 192.168.72.128;
		
		location / {
			root ip;
			index index.html;
		}
	}
	
	server {
		listen 80;
		server_name www.myhost1.com;
		
		location / {
			root myhost1;
			index index.html;
		}
	}
	
	server {
		listen 86;
		server_name www.myhost1.com;
		
		location / {
			root myhost1;
			index index_86.html;
		}
	}
	
	server {
		listen 80;
		server_name www.myhost2.com;
		
		location / {
			root myhost2;
			index index.html;
		}
	}
	
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```
