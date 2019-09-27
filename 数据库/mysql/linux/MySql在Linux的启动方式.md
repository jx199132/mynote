在Linux系统下，MySQL服务器通常有四种启动方式：mysqld守护进程启动，mysqld_safe启动，mysql.server启动，mysqld_multi多实例启动

## mysqld守护进程启动
信息只会从终端输出，而不是记录在错误日志文件中，适用于调试阶段

## mysqld_safe启动
mysqld_safe是一个启动脚本，该脚本会调用mysqld启动，如果启动出错，会将错误信息记录到错误日志中

mysqld_safe启动mysqld和monitor mysqld两个进程，这样如果出现mysqld进程异常终止的情况，mysqld_safe会重启mysqld进程。 



## mysql.server启动
mysql.server同样是一个启动脚本，调用mysqld_safe脚本。它的执行文件在$MYSQL_BASE/share/mysql/mysql.server 和 support-files/mysql.server


## mysqld_multi多实例启动