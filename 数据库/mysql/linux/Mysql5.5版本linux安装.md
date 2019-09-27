# 准备工具
下载地址 https://dev.mysql.com/get/Downloads/MySQL-5.5/mysql-5.5.61-linux-glibc2.12-x86_64.tar.gz



# 添加mysql用户

```
groupadd mysql
#将mysql用户加入到用户组
useradd -r -g mysql mysql
```

# 安装mysql

## 解压

```
tar xf mysql-5.5.61-linux-glibc2.12-x86_64.tar.gz 
```
ps : tar xf 不用输出解压日志详情

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

## mysql安装

### 配置文件
```
cp /usr/local/mysql/support-files/my-medium.cnf /etc/my.cnf
vim /etc/my.cnf

[mysqld]
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
interactive_timeout=10
wait_timeout=10
character-set-server=utf8
max_connections=1000
[mysql]
default-character-set=utf8

```

### 初始化安装

```
#安装mysql，设置mysql的安装目录，以及数据库存储目录（mysql配置文件也可以指定通过--defaults-file=file_name）
scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```

输出内容

```
Installing MySQL system tables...
181011 10:18:02 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
181011 10:18:02 [Note] /usr/local/mysql/bin/mysqld (mysqld 5.5.61) starting as process 13446 ...
OK
Filling help tables...
181011 10:18:02 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
181011 10:18:02 [Note] /usr/local/mysql/bin/mysqld (mysqld 5.5.61) starting as process 13453 ...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/local/mysql/bin/mysqladmin -u root password 'new-password'
/usr/local/mysql/bin/mysqladmin -u root -h localhost.localdomain password 'new-password'

Alternatively you can run:
/usr/local/mysql/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:
cd /usr/local/mysql ; /usr/local/mysql/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl
cd /usr/local/mysql/mysql-test ; perl mysql-test-run.pl

Please report any problems at http://bugs.mysql.com/
```

## 设置开机启动，添加权限，启动MySQL

```
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig --level 2345 mysqld on
chown mysql:mysql -R /usr/local/mysql/
service mysqld start
```

## 修改MySQL登录密码


```
/usr/local/mysql/bin/mysql -uroot -p
#5.5这个版本默认是没有密码的说以我们在登录的时候可以直接点回车就可以了

mysql>use mysql;
mysql>update user set password=passworD("root") where user='root';
mysql>flush privileges;
mysql>exit;

```

## 修改所有用户访问数据库权限

```
/usr/local/mysql/bin/mysql -uroot -p

mysql>use mysql;
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' identified by 'root'; 
mysql>flush privileges;
mysql>exit;  
```
