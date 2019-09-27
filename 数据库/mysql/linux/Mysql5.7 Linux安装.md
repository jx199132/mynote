
# 下载
注意下载的时候有几个选择，如下：

- mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz

- mysql-test-5.7.23-linux-glibc2.12-x86_64.tar.gz

- mysql-5.7.23-linux-glibc2.12-x86_64.tar

下载第一个即可，第三个等于第一个+第二个

下载地址：https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz

下载完毕之后放到/home/目录下



# 解压

ps:如果下载的第三个，那么解压命令为 tar -xvf ，解压出来会存在第一个和第二个

```
# cd /home/
# tar -zxvf mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz 
```
# 移动

```
# mv mysql-5.7.23-linux-glibc2.12-x86_64 /usr/local/
```
# 重命名

```
# mv /usr/local/mysql-5.7.23-linux-glibc2.12-x86_64 /usr/local/mysql
```
# 创建data文件夹

```
# mkdir /usr/local/mysql/data
```

# 新建mysql用户、mysql用户组

```
# groupadd mysql
# useradd mysql -g mysql
```
# 将/usr/local/mysql的所有者及所属组改为mysql

```
# chown -R mysql.mysql /usr/local/mysql
```

# 配置

```
# /usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data 
```

如果出现错误：

```
/usr/local/mysql/bin/mysqld: error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory

```

则执行

```
# yum -y install numactl
```
如果出现错误：

```
error while loading shared libraries: libaio.so.1
```
则执行


```
yum install  libaio-devel.x86_64
```

ubuntu

```
sudo apt-get install libaio-devel
```


完成后继续执行

```
# /usr/local/mysql/bin/mysqld --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data --initialize
```

记住生成的临时密码(eP0esi#go8Un)

```
2018-08-02T03:53:11.709504Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2018-08-02T03:53:12.018954Z 0 [Warning] InnoDB: New log files created, LSN=45790
2018-08-02T03:53:12.100635Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2018-08-02T03:53:12.157186Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 93b0bfb5-9607-11e8-9fce-52540045e46d.
2018-08-02T03:53:12.158285Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2018-08-02T03:53:12.158900Z 1 [Note] A temporary password is generated for root@localhost: eP0esi#go8Un

```


# 编辑/etc/my.cnf


```
# vim /etc/my.cnf
```

```
[mysqld]
#服务端口号 默认3306
port = 3306

#mysql数据文件所在位置
datadir=/usr/local/mysql/data

#mysql安装根目录
basedir=/usr/local/mysql

#设置socke文件所在目录
socket=/tmp/mysql.sock

#数据库默认字符集
character-set-server=utf8

# 错误日志地址
log-error=/usr/local/mysql/data/error.log

#pid文件所在目录
pid-file=/usr/local/mysql/data/mysqld.pid

```
# 将mysql加入服务开机自启

```
# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
```
进行编辑 /etc/init.d/mysql

vim /etc/init.d/mysql 将文件中 basedir和datadir改成如下

```

basedir=/usr/local/mysql
datadir=/usr/local/mysql/data

```

# 启动mysql

```
# service mysql start
```
停止用  service mysql stop


ps : ubuntu 下用 sudo /etc/init.d/mysql start

# mysql并且修改密码，开启远程访问（这一步完成之后就可以在其他机器通过客户端工具连接msyql了）

```
## 输入下面命令会提示输入密码，输入上面的临时密码(eP0esi#go8Un)即可
# /usr/local/mysql/bin/mysql -u root -p



mysql> set password=password('root2018');

mysql> grant all privileges on *.* to 'root'@'%' identified by 'root2018';

mysql> flush privileges;

```
# 添加系统路径

```
# vim /etc/profile

在profile文件最后面加上

export PATH=/usr/local/mysql/bin:$PATH


```
刷新环境变量

```
# source /etc/profile
```


# mysql加入开机自动启动

```
# chkconfig mysql on
```



# 导入数据出错，在/etc/my.cnf中加入


```
sql_mode = ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

```

