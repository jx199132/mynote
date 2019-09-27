# redis提供的5种数据结构
## String
可以是字符串，整数，浮点数
## list
一个链表，链表的每一个节点都包含一个字符串
## set
无序集合，包含不重复的字符串
##hash
包含键值对的无序散列表
## zset
有序集合
存储分数与元素之间的映射,元素排列顺序根据分值大小来决定

# 具体使用命令
## String
****字符串操作****
### get 
获取存储在给定键中的值 

```
get(key)
```

### set
设置存储在给定键中的值,如果已经存在将会覆盖

```
set(key)
```

### del
删除存储在给定键中的值(适用于redis提供的5种数据结构) 

```
del(key)
```

****数值操作****
### incr
在键存储的值上 自增1 ,如果不存在key，那么从0开始增加

```
incr(key)
```

### decr 
在键存储的值上 自减1 如果不存在key，那么从0开始减少

```
decr (key)
```

### incrby 
在键存储的值上 加上 整数 , 如果不存在key，那么从0开始增加

```
incrby (key,num)
```

### decrby
在键存储的值上 减去 整数, 如果不存在key，那么从0开始减少

```
decrby(key,num)
```

### incrbyfloat
在键存储的值上 加上 浮点数 (redis2.6及以上版本可用)

```
incrbyfloat(key,float)
```

****子串操作****
### append
在键存储的值上 做追加

```
append(key,appendStr)
```

### getrange
在键存储的值上 获取 一段范围类的子串

```
getrange(key,start,end)
```

### setrange
在键存储的值上 将从偏移量开始的子串设置为指定的值

```
setrange(key,offset,value)
```

****位操作，不常用****
### getbit
将字符串看做是二进制字位串,并返回位串中偏移量为offset的二进制值

```
getbit(key,offset)
```

### setbit
将字符串看做是二进制位串,并将位串中偏移量为offset的二进制值设置为value

```
setbit(key,offset,value)
```

### bitcount
统计二进制位串里面值为1的二进制位数量 

```
bitcount(key ,[start , end])
```

### bitop
对一个或者多个二进制位串执行包括and,or,xor,not在内的任意一种按位运算操作，并将计算获取的结果保存在destkey的键里面

```
bitop operation destkey key1,key2....
```

## list
****常用操作****
### rpush
将一个或者多个值，推入列表的右端

```
rpush(key,value...)
```

### lpush
将一个或者多个值，推入列表的左端

```
lpush(key,value...)
```

### rpop
移除并返回列表最右端的元素

```
rpop(key)
```

### lpop
移除并返回列表最左端的元素

```
lpop(key)
```

### lindex
返回列表中偏移量为offset的元素

```
lindex(key,offset)
```

### lrange
返回列表从start偏移量到end偏移量范围内的所有元素，偏移量start和end的都包含在内

```
lrange(key,start,end)
```

### ltrim
对列表进行修剪，只保留从start到end之内的元素，包含start和end

```
ltrim(key,start,end)
```

****阻塞操作****
### blpop
移除并返回列表最左端的元素，或者在timeout时间类阻塞并且等待元素出现

```
blpop(key,timeout)
```

### brpop
移除并返回列表最右端的元素，或者在timeout时间类阻塞并且等待元素出现

```
brpop(key,timeout)
```

****移动操作****
### rpoplpush
从srckey弹出最右端的元素，加入destkey做左端

```
rpoplpush(srckey,destkey)
```

### brpoplpush
从srckey弹出最右端的元素，加入destkey做左端,如果srckey为空那么等待timeout

```
brpoplpush(srckey,destkey,timeout)
```

## set
****单集合处理****
### sadd
将一个或者多个值添加到集合中，并返回被添加元素中不存在与集合中的数量

```
sadd(key,item...)
```

### srem
从集合中移除一个或者多个元素，并返回被移除的数量

```
srem(key,itme...)
```

### sismember
检查元素item是否存在于集合中

```
sismember(key,item)
```

### scard
返回集合包含的元素数量

```
scard(key)
```

### smembers
返回集合包含的所有元素

```
smembers(key)
```

### srandmember
从集合中随机返回一个或者多个元素，
当count不填写时，返回一个元素.
当count填写并且为正数，那么返回的元素个数为count，并且不重复
当count填写并且为负数，那么返回的元素个数为count绝对值，并且可能存在重复

```
srandmember(key,[count])
```

### spop
随机移除集合中的一个元素，并获取它

```
spop(key)
```

****多集合处理****
### smove
从src集合获取元素item，添加dest集合中. 如果src中存在item，那么返回 1 否则返回 0

```
smove(src,dest,item)
```
### sdiff
返回那些存在于第一个集合，但是不存在与其他集合中的元素

```
sdiff(keys...)
```

### sdiffstore
将那些存在于第一个集合，但是不存在与其他集合中的元素存储到dest中

```
sidffstore(dest,keys...)
```

### sinter
返回同时存在于所有集合中的元素

```
sinter(keys...)

```
### sintersotre
将那些所有集合都存在的元素，存储到dest中

```
sintersotre(dest,keys...)
```
### sunion
返回那些至少存在一个集合中的元素

```
sunion(kyes...)
```
### sunionstore
将那些至少存在于一个集合中的元素存储到dest中(实际上就是合并所有集合)

```
sunionstore(keys...)
```

## hash(键值对)
### hexists
检查指定的键是否存在于散列中

```
hexists(key1,key2) key1
```
**是散列的key，key2是散列中键值对的key**
### hkeys
获取散列中所有的键

```
hkeys(key)
```
### hvals
获取散列中所有的值

```
hvals(key)
```
### hgetall
获取散列包含的所有键值对

```
hgetall(key)
```
### hincrby
将散列中key存储的值加上整数

```
hincrby(key1,key2,num)
```
### hincrbyfloat
将散列中key存储的值加上浮点数

```
hincrbyfloat(key1,key2,float)
```

## zset（有序集合）
****常用命令****
### zadd
将带有指定分值的元素添加到有序集合中

```
zadd(key,score,item)
```
### zrem
从有序集合中移除指定的元素，并返回移除的数量

```
zrem(key,item...)
```
### zcard
返回有序集合包含的元素数量

```
zcard(key)
```
### zincrby
将元素的分值加上num

```
zincrby(key,num,item)
```
### zcount
返回分值介于min与max之间的数量

```
zcount(key,min,max)
```
### zrank
返回元素在有序集合中的排名

```
zrank(key,item)
```
### zscore
返回元素的分值

```
zscore(key,item)
```
### zrange
返回有序集合中排名介于start和stop之间的成员，如果withscores可选项存在，那么连带分数也会一起返回

```
zrange(key,start,stop,[withscores])
```

### zrevrank
返回有序集合中元素item的排名，按照分数从大到小

```
zrevrank(key,item)
```
### zrevrange
返回有序集合中给定排名范围内的元素，按照分值从大到下排列

```
zrevrange(key,start,stop,[withscores]
```
### zrangebyscore
返回有序集合中分值介于min与max中的元素

```
zrangebyscore (key,min,max,[withscores] [limit offset count]
```
### zrevrangebyscore
返回有序集合中分值介于min与max中的成员，并按照分值从大到小排序

```
zrevrangebyscore(key,max,min,[withscores] [limit offset count]
```
### zremrangebyrank
移除有序列表中排名介于start与stop中间的元素

```
zremrangebyrank(key,start,stop)
```
### zremrangebyscore
移除有序集合中分值介于min与max之间的元素

```
zremrangebyscore(key,min,max)
```
### zinterstore
将多个集合进行交集运算存储到dest中 (opt过滤选项)

```
zinterstore(dest,keys,[opt])
```
### zunionstore
将多个集合进行并集运算存储到dest中(opt过滤选项)

```
zunionstore(dest,keys,[opt])
```