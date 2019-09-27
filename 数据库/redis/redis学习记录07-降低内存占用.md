# 背景
一个redis服务器，内存是有限的，那么能够存储的内容也是有大小限制的.
# 解决方案
## 短结构
### 概念
redis为列表，集合，散列和有序集合提供了一组配置选项，可以让redis以更加解压空间的形式存储长度较短的结构，即成为 短结构.
### 配置选项
在redis配置文件中不同的结构都存在各自的配置

```
list-max-ziplist-entries 512
list-max-ziplist-value 64

hash-max-ziplist-enries 512
hash-max-ziplist-value 64

zset-max-ziplist-enries 512
zset-max-ziplist-value 64

set-max-ziplist-enries 512
set-max-ziplist-value 64
```
entries表示在使用压缩列表的情况下最多允许存储的元素数量为512
value 表示在使用压缩列表的情况下单个元素最多存储为64字节
当任意一个参数突破限制的时候，那么压缩列表的编码结果将被转换为其他的结果，而内存的占用也会增加
### 示例(以集合来举例说明，其他结果一样)
添加一个set 键 k1 ， 为k1中添加500条数据(0-499)
```
		String redisIp = "192.168.95.128";
        int reidsPort = 6379;
        JedisPool jedisPool = new JedisPool(new JedisPoolConfig(), redisIp, reidsPort);
        
        Jedis j = jedisPool.getResource();
        for(int i = 0 ; i < 500 ; i ++){
        	j.sadd("k1", i+"");
        }
```
查看k1 存在500条数据，k1 结果  ： encoding:intset serializedlength:1010
```
192.168.95.128:6379> scard k1
(integer) 500
192.168.95.128:6379> debug object k1
Value at:0x7f00d260f000 refcount:1 encoding:intset serializedlength:1010 lru:16275759 lru_seconds_idle:33
```
继续运行上面代码 在加入500条数据(500-999),注意把开始值结束值改一下
继续查看结果 发现encoding改成了hashtable，而且存储的空间变大了.
```
192.168.95.128:6379> scard k1
(integer) 1000
192.168.95.128:6379> debug object k1
Value at:0x7f00d260f000 refcount:1 encoding:hashtable serializedlength:2874 lru:16276017 lru_seconds_idle:1

```
### 结论
当一个结果突破到了为压缩列表设置的限制条件时，redis自动将它转换成更为典型的底层结构类型。
使用压缩列表：节约存储空间，但是每次存储和读取需要压缩与解压缩，当量一旦上来会产生性能问题.
