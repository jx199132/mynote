# 获取HTTPS证书

## 命令行生成

## 可视化阿里云购买
从阿里云购买 -- 管理控制台 -- SSL 证书（应用安全） -- 购买证书

根据需要来，这里用测试免费的为例

品牌：Symantec
选择证书类型：先选择增强型 -- 》 然后免费型 就会显示，  选择免费的

填写信息，进行购买即可。
然后进行下载，选择nginx


# 部署HTTPS证书

这里在nginx目录下新建了一个文件夹https_pem，将获取的证书放到该目录下

# 配置Nginx支持HTTPS
## 确认nginx是否支持ssl
确认nginx是否安装了ssl模块，如果没装会包错误

```
nginx: [emerg] unknown directive "ssl"
```


重新编译安装（ps：备份配置文件）

```
./configure --with-http_ssl_module
```


## 配置nginx

如下：

将80端口改成 443 ，因为http默认访问端口是 80 ， https默认访问端口是 443 

配置ssl_certificate（公钥路径），ssl_certificate_key（私钥路径）

其他ssl开头照搬即可

```
server {
    listen       443;
    ssl on;
    ssl_certificate /usr/local/nginx/https_pem/1726205_hp965.com.pem;
    ssl_certificate_key /usr/local/nginx/https_pem/1726205_hp965.com.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    server_name  www.hp965.com;
    location / {
        root   html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

重启nginx，输入https://www.hp965.com 即可发现跳转成功




## 设置HTTP强制跳转HTTPS
上一步成功之后可能会发现通过原来的http://www.hp965.com无法访问网页了，因为HTTP默认走的是80端口，我们刚才将其修改为443端口了。在这里我们可以在配置文件的最后一行加入以下代码：


```
server {
    listen 80;
    server_name www.hp965.com;  #这里修改为网站域名
    rewrite ^(.*)$ https://$host$1 permanent;
}
```


# 配置Nginx + tomcat支持HTTPS

## 配置Nginx


跟上面基本上一样，只是将location 配置了 tomcat的反向代理，然后加上upstream
```
server {
        listen       443;
        server_name  www.hp965.com;
		
		ssl on;
		ssl_certificate /usr/local/nginx/https_pem/1726205_hp965.com.pem;  #修改为fullchain.pem所在的路径
		ssl_certificate_key /usr/local/nginx/https_pem/1726205_hp965.com.key; # 修改为privkey.pem所在的路径
		ssl_session_timeout 5m;
		ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
		ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
		ssl_prefer_server_ciphers on;
		
		charset utf-8;

     	location / {
			proxy_pass        http://www.hp965.com;
			proxy_connect_timeout 5;
			proxy_redirect off;
			proxy_set_header   Host             $host;
			proxy_set_header   X-Real-IP        $remote_addr;
			proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
			<!--注意这里，如果不见会出问题-->
			<!--访问www.hp965.com 会跳转到 http://www.hp965.com:443-->
			proxy_set_header   X-Forwarded-Proto https;
		}

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
	
	server {
		listen 80;
		server_name www.hp965.com;  #这里修改为网站域名
		rewrite ^(.*)$ https://$host$1 permanent;
	}
	
	upstream  www.hp965.com
	{
		server   127.0.0.1:8080;
	}
```

## 配置tomcat

1 在修改端口的地方做调整

如下：加上了编码utf-8，将redirectPort改成443，加上proxyPort 等于443
```
 <Connector URIEncoding="utf-8" port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="443" proxyPort="443" />

```


2 在host下面加入一段，完整的贴出来了

```
<Valve className="org.apache.catalina.valves.RemoteIpValve"
          remoteIpHeader="x-forwarded-for"
          remoteIpProxiesHeader="x-forwarded-by"
          protocolHeader="x-forwarded-proto"/>
```



# 完整的配置（加入了静态资源映射）


## nginx

```
    server {
		listen 80;
		server_name www.hp965.com;  #这里修改为网站域名
		rewrite ^(.*)$ https://$host$1 permanent;
	}
	
	server {
		listen 443;
        server_name  www.hp965.com;
		
		ssl on;
		ssl_certificate /usr/local/nginx/https_pem/1726205_hp965.com.pem;  #修改为fullchain.pem所在的路径
		ssl_certificate_key /usr/local/nginx/https_pem/1726205_hp965.com.key; # 修改为privkey.pem所在的路径
		ssl_session_timeout 5m;
		ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
		ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
		ssl_prefer_server_ciphers on;
		
		charset utf-8;
		
		location  /css/ {
			root  /usr/local/nginx/myresources/static;
		}
		
		location  /ext/ {
			root  /usr/local/nginx/myresources/static;
		}
		
		location  /img/ {
			root  /usr/local/nginx/myresources/static;
		}
		
		location  /js/ {
			root  /usr/local/nginx/myresources/static;
		}
		
     	location / {
			proxy_pass        http://www.hp965.com;
			proxy_connect_timeout 5;
			proxy_redirect off;
			proxy_set_header   Host             $host;
			proxy_set_header   X-Real-IP        $remote_addr;
			proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
			proxy_set_header   X-Forwarded-Proto https;
		}

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
	
	
	
	upstream  www.hp965.com
	{
		server   127.0.0.1:8080;
	}
```


## tomcat

```
 <Connector URIEncoding="utf-8" port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="443" proxyPort="443" />


 <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        <!--新加入内容-->
        <Valve className="org.apache.catalina.valves.RemoteIpValve"
                  remoteIpHeader="x-forwarded-for"
                  remoteIpProxiesHeader="x-forwarded-by"
                  protocolHeader="x-forwarded-proto"
            />



      </Host>

```
