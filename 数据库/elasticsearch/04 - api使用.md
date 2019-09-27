
# 健康检查

通过postman 发送get请求即可
```
http://192.168.204.128:9200/_cat/health?v

epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1543302791 15:13:11  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```
# 节点查看

通过postman 发送get请求即可
```
http://192.168.204.128:9200/_cat/nodes?v

ip              heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.204.128            4          96   0    0.03    0.04     0.08 mdi       *      VUYEgAU
```

# 查看当前节点的所有Index

get 请求
```
http://192.168.204.128:9200/_cat/indices?v
```


# 新建索引

put请求

新建一个名为weather的索引

?pretty 表示以JSON格式化后的样子返回请求结果
```
http://192.168.204.128:9200/weather?pretty
```

返回下面，表示成功
```
{
    "acknowledged": true,
    "shards_acknowledged": true
}
```


# 删除索引
delete请求

```
http://192.168.204.128:9200/weather
```
