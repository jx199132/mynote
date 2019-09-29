# 环境 centos7

# 下载
http://nginx.org/download/nginx-1.13.0.tar.gz
# 解压

```
tar -zxvf nginx-1.13.0.tar.gz 
```
# 安装
安装nginx之前有几个必备软件在下面的列出来

gcc编译器（nginx不会直接提供二进制可执行程序，安装：yum install -y gcc）

pcre库（兼容正则表达式，在nginx.conf中就会有正则配置，安装：yum install -y pcre pcre-devel）

zlib库（nignx.conf 中如果开启gzip压缩就会用到，安装：yum install -y zlib  zlib-devel）

OpenSSL开发库（如果服务器不只是需要HTTP，还需要支持SSL  安装：yum install -y openssl openssl-devel）
```
cd nginx-1.13.0
./configure
make
make install
```
这里没有指定安装路径，如果需要指定路径可以
```
./configure --prefix=/usr/local/nginx
```



编译完毕之后可以看到输出内容 ，下面截取部分（这部分描述了nginx的安装路径）

```
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"

```

完成安装的目录结构 
配置文件路径 usr/local/nginx/conf/nginx.conf
启动二进制程序 usr/local/nginx/sbin/nginx

```
├── conf
│   ├── fastcgi.conf
│   ├── fastcgi.conf.default
│   ├── fastcgi_params
│   ├── fastcgi_params.default
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── mime.types.default
│   ├── nginx.conf
│   ├── nginx.conf.default
│   ├── scgi_params
│   ├── scgi_params.default
│   ├── uwsgi_params
│   ├── uwsgi_params.default
│   └── win-utf
├── html
│   ├── 50x.html
│   └── index.html
├── logs
└── sbin
    └── nginx

```

# 启动与关闭

## 启动
```
/usr/local/nginx/sbin/nginx  #默认选择配置文件 /usr/local/nginx/conf/nginx.conf
/usr/local/nginx/sbin/nginx -c path #指定配置文件
```

## 设置开机启动


```
vim /etc/rc.local
#添加一行：
/usr/local/nginx/sbin/nginx
#设置执行权限
cd /etc/
chmod 755 rc.local
```


## 查看nginx信息

```
/usr/local/nginx/sbin/nginx -t  #不启动nginx的情况下 查看配置文件是否有错误
/usr/local/nginx/sbin/nginx -v  #查看nginx版本信息
```
## 停止服务

```
 /usr/local/nginx/sbin/nginx -s stop #快速停止 杀死进程
 /usr/local/nginx/sbin/nginx -s quit #优雅停止 停止监听端口，处理完毕手上处理的链接在退出
```
## 重新读取配置文件并重新启动

```
/usr/local/nginx/sbin/nginx -s reload
```

## 查看帮助

```
/usr/local/nginx/sbin/nginx -h
```


# 卸载

```
删除nginx目录即可

rm -rf /opt/nginx

(rm -rf /usr/local/nginx)

如果配置了自启动，也需要删除
vim /etc/rc.local
在里面删掉 nginx这一行 
```




