# 环境准备
## 系统linux centos7两台
- master 114.116.86.168
- slave 154.8.194.61
## mysql版本 mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz
- 端口都为3306
- 密码都为root2018，对外开放连接

# 主服务器配置

- 编辑配置文件
vim /etc/my.cnf

```
#启用二进制日志
log-bin=mysql-bin   
#服务器唯一ID，默认是1，一般取IP最后一段,保证不重复即可
server-id=168
```
- 重启mysql

```
# service mysql restart
```
- 建立帐户并授权slave

```
mysql> GRANT REPLICATION SLAVE ON *.* to 'mysync'@'%' identified by 'q123456';
```
如果出现异常：

```
[Err] 1055 - Expression #1 of ORDER BY clause is not in GROUP BY clause and contains nonaggregated column 'information_schema.PROFILING.SEQ' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```
则编辑配置文件

vim /etc/my.cnf

[mysqld]

sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

重启服务

service mysql restart

重新执行建立账户授权slave语句

- 查询master的状态

```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000004 |      314 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+


```

记录下mysql-bin.000004 和 314

注：执行完此步骤后不要再操作主服务器MYSQL，防止主服务器状态值变化

# 配置从服务器Slave
- 编辑配置文件
vim /etc/my.cnf

```
#启用二进制日志
log-bin=mysql-bin   
#服务器唯一ID，默认是1，一般取IP最后一段,保证不重复即可
server-id=61

#如果主机器跟从机器一样，主机器出现异常，那么从机器直接加上，因为肯定也会有该异常
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```
- 重启mysql

- 指定主服务器master

```
#注意不要断开，314数字前后无单引号
mysql> change master to master_host='114.116.86.168',master_user='mysync',master_password='q123456',master_log_file='mysql-bin.000004',master_log_pos=314;
```
如果出现同主服务器一样的异常，解决方式一样

- 启动从服务器复制功能

```
mysql> start slave;
```

注：取消主从复制

stop slave;

reset slave;
- 检查从服务器复制功能状态

```
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 114.116.86.168
                  Master_User: mysync
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 314
               Relay_Log_File: VM_0_16_centos-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 314
              Relay_Log_Space: 536
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 168
                  Master_UUID: cab3cd5b-961f-11e8-8cab-fa163e2bbbae
             Master_Info_File: /usr/local/mysql/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 

```
注：Slave_IO_Running及Slave_SQL_Running进程必须正常运行，即YES状态，否则都是错误的状态(如：其中一个NO均属错误)。

# 测试
- 在主服务器创建一个数据库，并且新建一张表，完成一些数据变动，看看从服务器是不是自动也相应的跟主服务器保持同步

# 扩展
- 需要加入一台新的机器当做从服务器

但是主服务器做了一些操作，但是需要保证新从机器与旧从机器一致

那么重复从服务器的操作即可，把二进制文件和Position 设置的跟旧从机器当前的语句一样，那么也可以把前面的数据都同步过来

- 从服务器关机了，主服务器继续操作，当从服务器启动之后，关机期间主服务的操作会自动同步过来
