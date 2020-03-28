# Docker命令

启动、重启、关闭

```shell
service docker start
service docker restart
service docker stop

或者

systemctl start docker
systemctl restart docker
systemctl stop docker
```





# 安装redis

## 检索镜像

```shell
#	这里redis就是镜像名
docker search redis
```

## 下载镜像

```shell
#	这里redis就是镜像名
docker pull redis
```

## 查看镜像列表

```shell
# 下面的 Image Id 就是镜像的 id
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              de25a81a5a0b        2 weeks ago         98.2 MB

```



## 删除镜像

```shell
# 删除镜像 参数 的 镜像 ID
[root@localhost ~]# docker rmi de25a81a5a0b
```



## 创建一个新的容器并运行一个命令

```shell
#	使用docker镜像redis以后台模式启动一个容器,并将容器命名为test_redis
[root@localhost ~]# docker run --name test_redis -d redis
bd5df8da0d86a5753ac6b5441e2c85231b8e60d578f70a0798c45777f484e113
```

- --name : 为容器取名字，这里名字为 test_redis
- -d : 执行完这句命令之后，控制台不被阻碍，可以继续输入命令
- Redis 是镜像的名称



**ps：执行完毕之后会生成一段长字符串，一般可以用来做日志文件的名称，另外下面查看容器列表的命令CONTAINER ID这一段的值会是该值的前面一段**



## 查看容器列表

```
[root@localhost ~]# docker ps
```

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191031203847.png)

- CONTAINER ID : 启动时生成的ID 

- IMAGE ： 该容器使用的镜像
- COMMAND : 容器启动时调用的命令
- CREATED ： 容器的创建时间
- STATUS ： 容器当前状态
- PORTS : 容器系统使用的端口号



## 查看运行和停止状态的容器列表

```shell
docker ps -a
```



## 容器启动与停止以及删除

```shell
docker start 容器名/容器ID
docker stop 容器名/容器ID
docker rm 容器名/容器ID
```



## 端口映射

在docker容器中运行的软件所使用的端口，在本机或者局域网是不能访问的，需要把docker容器中的端口映射到当前主机的端口上，这样才能访问。

在docker中端口是6379 ， 映射到本机 6378

```shell
docker run -d -p 6378:6379 --name port_redis redis
```

这里可以看到有一个转换

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191031204339.png)



## 登录docker



```shell
# port_redis ： 是容器的名称
docker exec -it port_redis bash
```

登录之后可以像访问linux系统一样使用一些命令。 通过exit命令退出docker





# 安装Nginx

## 下载镜像

```shell
# 下载镜像
docker pull nginx
```

## 简单使用：创建一个容器

```shell
# 使用 NGINX 默认的配置来启动一个 Nginx 容器实例
docker run --name nginx-test -p 8081:80 -d nginx
```

- `nginx-test` 容器名称。

- `-d`设置容器在在后台一直运行。

- `-p` 端口进行映射，将本地 8081 端口映射到容器内部的 80 端口

  这时 输入 http://172.16.72.150:8081/ 即可访问 容器中的 nginx 了， 



## 了解使用：再创建一个容器(第二个容器)

```shell
# ps:上面的命令可以继续使用，这样就根据 nginx镜像启动了 两个 容器，它们之间互不干扰
docker run --name nginx-test2 -p 8082:80 -d nginx
```

这时 输入 http://172.16.72.150:8082/ 即可访问 容器中的 nginx 了



### 配置第二个容器nginx和宿主机器的映射

#### 在宿主机器创建目录 nginx

```shell
mkdir -p /home/nginx/www /home/nginx/logs /home/nginx/conf
```

- **www**: 目录将映射为 nginx 容器配置的虚拟目录
- **logs**: 目录将映射为 nginx 容器的日志目录
- **conf**: 目录里的配置文件将映射为 nginx 容器的配置文件



#### 拷贝容器内的文件到宿主机器

拷贝容器内 Nginx 默认配置文件到本地当前目录下的 conf 目录，容器 ID 可以查看 **docker ps** 命令输入中的第一列（这里的id是 容器nginx-test2的id）

```shell
# ps 如果不知道容器中 nginx 几个重要文件路径 可以 登录 docker 通过 whereis命令查找
docker cp 420c4c555b4e:/etc/nginx/nginx.conf /home/nginx/conf
```



同样拷贝将html文件拷贝出来

```shell
docker cp 420c4c555b4e:/usr/share/nginx/html/ /home/nginx/www/
```



最后宿主机器的文件

```shell
nginx
├── conf
│   └── nginx.conf
├── logs
└── www
    └── html
        ├── 50x.html
        └── index.html

```



#### 拷贝宿主机器内容到 容器内

修改一下 拷贝到宿主机器的文件，然后拷贝到 容器内

```shell
# 修改 宿主机器 的 index.html
vim /home/nginx/www/html/index.html
# 复制 宿主机器的 index.html  到 容器内
docker cp /home/nginx/www/html/index.html 420c4c555b4e:/usr/share/nginx/html/index.html
```

重新访问浏览器 http://172.16.72.150:8082/ 可以看到 浏览器内容变了

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191101094809.png)







## 推荐使用：还创建一个容器（第三个容器）

直接创建一个容器，并且建立映射



```shell
docker run -d -p 8083:80 --name nginx-test3 -v /home/nginx/www/html:/usr/share/nginx/html -v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/nginx/logs:/var/log/nginx nginx
```

- **-p 8082:80：** 将容器的 80 端口映射到主机的 8082 端口。
- **--name nginx-test3：**将容器命名为 runoob-nginx-test3。
- **-v /home/nginx/www/html:/usr/share/nginx/html：**将我们自己创建的 www 目录挂载到容器的 /usr/share/nginx/html。
- **-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf：**将我们自己创建的 nginx.conf 挂载到容器的 /etc/nginx/nginx.conf。

**-v /home/nginx/logs:/var/log/nginx：**将我们自己创建的 logs 挂载到容器的 /var/log/nginx



这里挂载之后，修改宿主机器的文件，自动会修改  容器内的文件



