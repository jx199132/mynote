# 开启web管理界面

```
cd /usr/local/rabbitmq/sbin/
./rabbitmq-plugins enable rabbitmq_management
```

# 配置防火墙 ， 默认端口 15672（记得重启,忘记重启坑了好久）

# 重启rabbitmq

```
rabbitmqctl stop
rabbitmq-server -detached
```
