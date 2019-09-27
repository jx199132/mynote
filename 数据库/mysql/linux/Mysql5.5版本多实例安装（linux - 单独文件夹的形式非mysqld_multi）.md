# 准备工具
下载地址 https://dev.mysql.com/get/Downloads/MySQL-5.5/mysql-5.5.61-linux-glibc2.12-x86_64.tar.gz

# 添加mysql用户

```
groupadd mysql
#将mysql用户加入到用户组
useradd -r -g mysql mysql
```

## mysql安装目录构建

```
cd mysql-5.5.61-linux-glibc2.12-x86_64/
mv /home/download/mysql-5.5.61-linux-glibc2.12-x86_64 /usr/local/mysql
```

## 最终查看mysql安装目录

```
cd /usr/local/mysql

ls -all

```
输出

```
总用量 56
drwxr-xr-x. 13 root root    213 10月 11 09:28 .
drwxr-xr-x. 16 root root    179 10月 11 09:38 ..
drwxr-xr-x.  2 root root   4096 10月 11 09:28 bin
-rw-r--r--.  1 7161 31415 17987 6月  16 00:34 COPYING
drwxr-xr-x.  3 root root     18 10月 11 09:28 data
drwxr-xr-x.  2 root root     55 10月 11 09:28 docs
drwxr-xr-x.  3 root root   4096 10月 11 09:28 include
-rw-r--r--.  1 7161 31415   301 6月  16 00:34 INSTALL-BINARY
drwxr-xr-x.  3 root root   4096 10月 11 09:28 lib
drwxr-xr-x.  4 root root     30 10月 11 09:28 man
drwxr-xr-x. 10 root root   4096 10月 11 09:28 mysql-test
-rw-r--r--.  1 7161 31415  2496 6月  16 00:34 README
drwxr-xr-x.  2 root root     30 10月 11 09:28 scripts
drwxr-xr-x. 27 root root   4096 10月 11 09:28 share
drwxr-xr-x.  4 root root   4096 10月 11 09:28 sql-bench
drwxr-xr-x.  2 root root   4096 10月 11 09:28 support-files
```


## 权限赋予

```
#将mysql目录及子目录归属给mysql用户
chown -R mysql /usr/local/mysql/
#将mysql目录及子目录归属给mysql用户组
chgrp -R mysql /usr/local/mysql/
```


# 多实例安装开始

## 创建实例目录

```
mkdir -p /data/3306
mkdir -p /data/3307
```

## 创建实例配置文件

```
cp /usr/local/mysql/support-files/my-medium.cnf /data/3306/my.cnf
cp /usr/local/mysql/support-files/my-medium.cnf /data/3307/my.cnf
```

## 创建实例数据存储目录

```
mkdir -p /data/3306/data
mkdir -p /data/3307/data
```

## 编辑3306实例配置文件

```
[client]
port = 3306
socket = /data/3306/mysql.sock
[mysqld]
port = 3306
socket = /data/3306/mysql.sock
basedir = /application/mysql-5.5.32
datadir = /data/3306/data
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
skip-name-resolve
log-bin=mysql-bin
binlog_format=mixed
max_binlog_size = 500M
server-id = 1
[mysqld_safe]
log-error=/data/3306/ilanni.err
pid-file=/data/3306/ilanni.pid
[mysqldump]
quick
max_allowed_packet = 16M
[mysql]
no-auto-rehash
[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M
[mysqlhotcopy]
interactive-timeout
```

## 编辑3307实例配置文件

```
[client]
port = 3307
socket = /data/3307/mysql.sock


[mysqld]
port = 3307
socket = /data/3307/mysql.sock
basedir = /usr/local/mysql
datadir = /data/3307/data


skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
skip-name-resolve
log-bin=mysql-bin
binlog_format=mixed
max_binlog_size = 500M
server-id = 2


[mysqld_safe]
log-error=/data/3307/ilanni.err
pid-file=/data/3307/ilanni.pid


[mysqldump]
quick
max_allowed_packet = 16M


[mysql]
no-auto-rehash
[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M
[mysqlhotcopy]
interactive-timeout
```

ps：3306 和 3307的配置文件 主要区别在与 路径 变化 与  server-id不能重复


## 分别初始化实例

```
/usr/local/mysql/scripts/mysql_install_db --defaults-file=/data/3306/my.cnf --user=mysql --basedir=/usr/local/mysql --datadir=/data/3306/data

/usr/local/mysql/scripts/mysql_install_db --defaults-file=/data/3307/my.cnf --user=mysql --basedir=/usr/local/mysql --datadir=/data/3307/data
```


## 分别启动实例

```
/usr/local/mysql/bin/mysqld_safe --defaults-file=/data/3306/my.cnf --user=root &

/usr/local/mysql/bin/mysqld_safe --defaults-file=/data/3307/my.cnf --user=root &
```

ps: 启动会输出内容完毕之后控制台会占用，敲回车即可，继续使用控制台，不会影响服务使用


## 分别连接实例

```
/usr/local/mysql/bin/mysql -P 3306 -uroot -p -S /data/3306/mysql.sock

/usr/local/mysql/bin/mysql -P 3307 -uroot -p -S /data/3307/mysql.sock
```

## 分别修改连接方式，密码

### 修改MySQL登录密码


```
/usr/local/mysql/bin/mysql -uroot -p
#5.5这个版本默认是没有密码的说以我们在登录的时候可以直接点回车就可以了

mysql>use mysql;
mysql>update user set password=passworD("root") where user='root';
mysql>flush privileges;
mysql>exit;

```

### 修改所有用户访问数据库权限

```
/usr/local/mysql/bin/mysql -uroot -p

mysql>use mysql;
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' identified by 'root'; 
mysql>flush privileges;
mysql>exit;  
```

## 分别关闭实例

```
/usr/local/mysql/bin/mysqladmin -uroot -p -S /data/3306/mysql.sock shutdown

/usr/local/mysql/bin/mysqladmin -uroot -p -S /data/3307/mysql.sock shutdown
```


