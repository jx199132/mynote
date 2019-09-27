
# 环境说明
centos7、redis3.2.8

# 安装GCC

```
yum -y install gcc
```

# 下载

```
wget -q http://download.redis.io/releases/redis-3.2.8.tar.gz
```
# 解压

```
tar -xf redis-3.2.8.tar.gz
```

# 目录移动&切换

```
mv  redis-3.2.8 /usr/local/redis
cd /usr/local/redis/
[root@localhost redis]# ls
00-RELEASENOTES  CONTRIBUTING  deps     Makefile   README.md   runtest          runtest-sentinel  src    utils
BUGS             COPYING       INSTALL  MANIFESTO  redis.conf  runtest-cluster  sentinel.conf     tests

```

# 安装Redis

- make
```
make
```
如果出现下面错误用 make MALLOC=libc 代替 make

```
jemalloc/jemalloc.h：没有那个文件或目录
```
- make install

```
make install
```

# 启动Redis

```
redis-server redis.conf 
```


# 连接redis

```
redis-cli -p 端口号
```

# 关闭Redis

到redis节点目录下执行如下命令

```
redis-cli -p 端口号 shutdown
```

获取通过客户端连接上redis，直接输入 shutdown

