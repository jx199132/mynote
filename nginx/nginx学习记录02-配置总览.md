# 基本配置
## 用于调试定位问题的配置
### daemon on|off
是否以守护进程的方式运行nginx，默认on

### master_process on|off
是否以master/worker的方式工作

### error_log path level
默认 error_log logs/error.log error
path代表的是一个具体的日志存储文件地址，可以是相对的也可以是绝对的
level 是日志的级别,取值范围debug，info，notice，warn，error，crit，alert，emerg
当设置为 error级别时 ，那么   error，crit，alert，emerg 都会输出到日志中
如果配置级别为debug，那么所有的日志都会输出
### debug_connection [IP|CIDR]
仅对指定的客户端输出debug级别的日志
这个配置属于事件性配置，需要放到events中才有效，要确保configure时 加入了 --with-debug参数.
events{
  debug_connection  192.168.1.1;
    debug_connection  192.168.1.2;
}
## 正常运行的配置
### include path
include配置项可以引入其他的配置文件到当前配置文件中，path可以是相对地址，也可以是绝对地址，也可以是通配符
include vhost/*.conf;

### pid path
pid文件的存放路径

### user username [groupname]
默认 user nobody nodoby
nginx worker进程运行的用户及用户组
## 优化性能配置项
### worker_processes number
nginx worker进程个数 默认 worker_processes 1;

### worker_cpu_affinity cpumask [cpumask...]
绑定nginx workder进程到指定的cpu内核

worker_processes 和 worker_cpu_affinity用法
```
http://www.360doc.com/content/12/0713/14/4330887_223977070.shtml
```

## 事件类配置(需要放到events块中)
### accept_mutex [on|off]
负载均衡锁（accept锁），默认是 on开启的， 当一个worker进程建立的连接数达到worker_connections配置的最大连接数7/8时，会大大的减少该worker进程试图建立新的tcp机会，来实现worker进程之上处理客户端请求数尽量相近，可以关闭它，那么建立tcp连接的时间会减少，但是worker进程之间的负责会不均衡.
### lock_file path
lock文件路径，如果accept锁关闭，那么此配置不生效。如果打开了锁，并且由于编译程序或者操作系统的因素导致nginx不支持原子锁，这时候才会使用文件锁来实现appect锁，这时候这个配置的lock文件才会生效


# http模块配置

## server块
由于IP地址的数量有限，因此经常存在多个主机域名对应同一个ip的情况，这时在nginx.conf中就可以按照server_name（对应用户请求中的主机域名）通过server块来定义虚拟主机.
每一个server块都是一个虚拟主机，它只处理与之相对应的主机域名请求


### listen
监听端口
配置块 server
### server_name
主机名称  nginx根据请求的header中取出host，与每个server中的server_name进行匹配，以此来决定哪个server来处理这个请求
配置块 server
### location
配置块 server
location会根据请求uri来匹配 与自身配置的uri进行匹配，如果可以匹配，就选择location块中的配置来处理用户请求.
语法：location [=|~|~*|^~|@]/uri

= ：把uri当做字符串，做完全匹配
~ ：表示匹配uri时大小写是敏感的
~*：表示匹配时忽略大小写敏感
^~：表示匹配uri时只需要前半部分满足uri即可 例如  ^~/images/     那么以 /images/开始的请求都会匹配上

### 文件路径定义
1  root方式设置资源路径
语法  root path
默认 root html
配置块 http 、server 、 location 、 if
例如 定义 资源相对于 http请求的目录
location /download/ {
  root /opt/web/html
}
如果请求uri是  /download/index/test.html ，那么 web服务器将会返回  /opt/web/html/download/index/test.html

2  alias 方式设置资源路径
配置块 location
例如 ： 若有有一个请求 uri 是 /conf/nginx.conf ，而 用户实际想访问的文件在 /usr/local/nginx/conf/nginx.conf 
 alias方式
    localtion /conf{
      alias /usr/local/nginx/conf;
   }
  root方式
   location /conf{
     root /usr/local/nginx;
  }
总结：root 用于定义 文件的根目录， 然后  根目录  + 请求的路径 进行拼接组合
          alias 在请求的uri 路径中 已经把/conf 这段字符串丢掉  最终 配置的目录 + uri替换掉/conf的字符串


### index
访问首页
配置块  http  、server 、 location

location / {
   root path;
   index /index.html /index.jsp /index.php
}
请求过来之后是域名path，这时候nginx会返回 path/index.html 中的结果，如果没有获取到 就继续 path/index.jsp

### error_page
配置块 http 、server、location、if
根据http返回码重定向到页面
例如  
error_page 404 /404.html
error_page 502 503 504 /50x.html

## 负载均衡配置
### upstream
upstream块定义了一个上游服务器集群，便于反向代理中的proxy_pass使用。例如：

```
upstream upname{
 server host1;
 server host2;
 server host3;
}
server{
 localtion /{
   proxy_pass http://upname;
 }
}
```

### upstream块中的server
server配置指定了一台上游服务器的名字，这个名字可以是域名，ip地址端口
在server后面还可以加上一些参数
weight=number： 设置这台上游服务器转发的权重，默认值1
max_fails=number：该选项与fail_timeout配合使用，在fail_timeout时间段内，如果向当前上游服务器转发失败次数超过number，则认为在当前的fail_timeout时间段内这台上游服务器不可用。max_fail默认值1 ，如果设置0，则不检查失败次数
fail_timeout=time：表示在该时间段内转发失败次数得到多少之后就认为该上游服务器暂时不可用.
down：表示所在的上游服务器永久下线，只有在使用ip_hash配置时这个配置才有效
backup：在使用ip_hash配置时它的配置是无效的。它的配置表示所在的上游服务器只是备份服务器，只有在所有的非备份上游服务器都失效之后，才会转发给这台上游服务器
### upstream块中的ip_hash
某些场景下，希望来自同一个用户的请求始终固定在一个上游服务器上，可以用它。
ip_hash根据客户端的ip计算出来一个值，然后根据上游服务器的个数 value%number获取一个固定值

```
upstream upname{
 ip_hash;
 server host1;
 server host2 down;
 server host3;
}
```

## 反向代理的基本配置
### proxy_pass
配置在 location 、if中
此配置将当前请求反向代理到URL参数指定的服务器上，URL可以是主机名、IP

```
proxy_pass http://www.exp.com
proxy_pass http://192.168.1.1:8089
```

还可以是upstream块

```
upstream upname{
  ...
}
server{
  location /{
    proxy_pass http://upname;
  }
}
```
默认情况下反向代理是不会转发请求中的host头部，如果需要转发，那么必须加上配置

```
		location / {
			proxy_pass  http://localhost:4019;
			proxy_set_header Host $host;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		}
```

### proxy_next_upstream
语法 `proxy_next_upstream[error|timeout|invalid_header|http_500|http_502|http_503|http_504|http_404|off]`
默认值 proxy_next_upstream error timeout
配置块 http、server、location
error：当向上游服务器发起连接，发起请求，读取响应时出错
timeout：发送请求或者读取响应超时
invaild_header：上游服务器发送响应是不合法的
http_xxx：上游服务器返回的HTTP响应时xxx
off：关闭proxy_next_upstream功能一出错就选择另一台上游服务器再次转发。

### proxy_connect_timeout
proxy_connect_timeout time
连接超时时间设置，如果这个时间内没有连接成功就认为连接不上，马上换另外一台上游服务器

代理的更多参数  参考

```
http://nginx.org/en/docs/http/ngx_http_proxy_module.html
```

# 完整的nginx.conf默认配置
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