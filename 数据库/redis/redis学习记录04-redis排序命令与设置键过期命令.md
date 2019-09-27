# 排序命令
SORT key [BY pattern] [LIMIT start count] [GET pattern] [ASC|DESC] [ALPHA] [STORE dstkey]  

转摘至 http://www.cnblogs.com/redcreen/archive/2011/02/15/1955226.html

## sort key 
这个是最简单的情况，没有任何选项就是简单的对集合自身元素排序并返回排序结果.下面给个例子

```
redis> lpush ml 12
(integer) 1
redis> lpush ml 11
(integer) 2
redis> lpush ml 23
(integer) 3
redis> lpush ml 13
(integer) 4
redis> sort ml
1. "11"
2. "12"
3. "13"
4. "23"
```
## [ASC|DESC] [ALPHA] 
sort默认的排序方式（asc）是从小到大排的,当然也可以按照逆序或者按字符顺序排。逆序可以加上desc选项，想按字母顺序排可以加alpha选项，当然alpha可以和desc一起用。下面是个按字母顺序排的例子
```
redis> lpush mylist baidu
(integer) 1
redis> lpush mylist hello
(integer) 2
redis> lpush mylist xhan
(integer) 3
redis> lpush mylist soso
(integer) 4
redis> sort mylist
1. "soso"
2. "xhan"
3. "hello"
4. "baidu"
redis> sort mylist alpha
1. "baidu"
2. "hello"
3. "soso"
4. "xhan"
redis> sort mylist desc alpha
1. "xhan"
2. "soso"
3. "hello"
4. "baidu"
```
## [BY pattern] 
除了可以按集合元素自身值排序外，还可以将集合元素内容按照给定pattern组合成新的key，并按照新key中对应的内容进行排序。下面的例子接着使用第一个例子中的ml集合做演示：  

```
redis> set name11 nihao
OK
redis> set name12 wo
OK
redis> set name13 shi
OK
redis> set name23 lala
OK
redis> sort ml by name* 
1. "13"
2. "23"
3. "11"
4. "12"
```
*代表了ml中的元素值，所以这个排序是按照name12 name13 name23 name23这四个key对应值排序的,当然返回的还是排序后ml集合中的元素 
(对排序结果有疑问可看最下面的FAQ)
## [GET pattern] 
上面的例子都是返回的ml集合中的元素。我们也可以通过get选项去获取指定pattern作为新key对应的值。看个组合起来的例子 

```
redis> sort ml by name* get name*  alpha
1. "lala"
2. "nihao"
3. "shi"
4. "wo"
```
这 次返回的就不在是ml中的元素了，而是name12 name13 name23 name23对应的值。当然排序是按照name12 name13 name23 name23值并根据字母顺序排的。另外get选项可以有多个。看例子（#特殊符号引用的是原始集合也就是ml）

```
redis> sort ml by name* get name* get #  alpha
1. "lala"
2. "23"
3. "nihao"
4. "11"
5. "shi"
6. "13"
7. "wo"
8. "12"
```
最后在还有一个引用hash类型字段的特殊字符->，下面是例子

```
redis> hset user1 name hanjie
(integer) 1
redis> hset user11 name hanjie
(integer) 1
redis> hset user12 name 86
(integer) 1
redis> hset user13 name lxl
(integer) 1
redis> sort ml get user*->name
1. "hanjie"
2. "86"
3. "lxl"
4. (nil)
```
很容易理解，注意当对应的user23不存在时候返回的是nil 
## [LIMIT start count] 
上面例子返回结果都是全部。limit选项可以限定返回结果的数量。例子

```
redis> sort ml get name* limit 1 2
1. "wo"
2. "shi"
```
start下标是从0开始的，这里的limit选项意思是从第二个元素开始获取2个 
##[STORE dstkey]  
如果对集合经常按照固定的模式去排序，那么把排序结果缓存起来会减少不少cpu开销.使用store选项可以将排序内容保存到指定key中。保存的类型是list

```
redis> sort ml get name* limit 1 2 store cl
(integer) 2
redis> type cl
list
redis> lrange cl 0 -1
1. "wo"
2. "shi"
```
这个例子我们将排序结果保存到了cl中
      功能介绍完后，再讨论下关于排序的一些问题。如果我们有多个redis server的话，不同的key可能存在于不同的server上。比如name12 name13 name23 name23，很有可能分别在四个不同的server上存贮着。这种情况会对排序性能造成很大的影响。redis作者在他的blog上提到了这个问题的解 决办法，就是通过key tag将需要排序的key都放到同一个server上 。由于具体决定哪个key存在哪个服务器上一般都是在client端hash的办法来做的。我们可以通过只对key的部分进行hash.举个例子假如我们 的client如果发现key中包含[]。那么只对key中[]包含的内容进行hash。我们将四个name相关的key，都这样命名[name]12 [name]13 [name]23 [name]23，于是client 程序就会把他们都放到同一server上。不知道jredis实现了没。 
       还有一个问题也比较严重。如果要sort的集合非常大的话排序就会消耗很长时间。由于redis单线程的，所以长时间的排序操作会阻塞其他client的 请求。解决办法是通过主从复制机制将数据复制到多个slave上。然后我们只在slave上做排序操作。并进可能的对排序结果缓存。另外就是一个方案是就 是采用sorted set对需要按某个顺序访问的集合建立索引。
实例：

```
redis> sadd tom:friend:list  123 #tom的好友列表 里面是好友的uid 
1 
redis> sadd tom:friend:list  456 
1 
redis> sadd tom:friend:list  789 
1 
redis> sadd tom:friend:list  101 
1 
redis> set uid:sort:123 1000 #uid对应的成绩 
OK 
redis> set uid:sort:456 6000 
OK 
redis> set uid:sort:789 100 
OK 
redis> set uid:sort:101 5999 
OK 
redis> set uid:123 "{'uid':123,'name':'lucy'}" #增加uid对应好友信息 
OK 
redis> set uid:456 "{'uid':456,'name':'jack'}" 
OK 
redis> set uid:789 "{'uid':789,'name':'marry'}" 
OK 
redis> set uid:101 "{'uid':101,'name':'icej'}"  
OK 
redis> sort tom:friend:list by uid:sort:* get uid:* #从好友列表中获得id与uid:sort字段匹配后排序,并根据排序后的顺序，用key在uid表获得信息 
1. {'uid':789,'name':'marry'} 
2. {'uid':123,'name':'lucy'} 
3. {'uid':101,'name':'icej'} 
4. {'uid':456,'name':'jack'} 
redis> sort tom:friend:list by uid:sort:* get uid:* get uid:sort:* 
1. {'uid':789,'name':'marry'} 
2. 100 
3. {'uid':123,'name':'lucy'} 
4. 1000 
5. {'uid':101,'name':'icej'} 
6. 5999 
7. {'uid':456,'name':'jack'} 
8. 6000
 
```
## FAQ:
1.sort ml by name* get name* get # 为什么会按照shi lala nihao wo的顺序排下来，这个跟单纯的排序name*和name * alpha的结果都不一样
    这个问题要从redis的实现逻辑上来分析了
    a)list在插入后 ,默认是按照时间的先后反序排列的 , lrange ml 0 -1,结果是:13 23 11 12. 这是因为list插入时是将最新的item插入到链表头
    b)sort m1 by name* 确定是会按照name*的值进行排序的.但当name*对应的value不是num型并且没有设置alpha的时候,会导致排序分值都是相同的,因为程序将把name*对应的值尝试转换为nun型
    c)这就会导致sort ml by name*会按照ml的自然顺序进行排列了
```
if (alpha) {
if (sortby) vector[j].u.cmpobj = getDecodedObject(byval);
} else {
if (byval->encoding == REDIS_ENCODING_RAW) {
    vector[j].u.score = strtod(byval->ptr,NULL);
} else if (byval->encoding == REDIS_ENCODING_INT) {
    /* Don't need to decode the object if it's
     * integer-encoded (the only encoding supported) so
     * far. We can just cast it */
    vector[j].u.score = (long)byval->ptr;
} else {
    redisAssert(1 != 1); 
}   
}   
```

# 设置key过期命令
存储在redis中的数据，可以通过del显示的删除.
也可以通过redis的过期时间来设置，在某一个时间之后自动删除
在redis中 只能够对 整个key设置过期时间，对于list，set，hash，zset来说，无法设置其中的单独子元素的过期时间.
## persist
移除键的过期时间

```
persist(key)
```
## ttl
查看键距离过期还有多少秒

```
ttl(key)
```
## expire
让给定的键在指定的秒之后过期

```
expire(key,seconds)
```
##expireat
让给定的键在指定的unix时间梭之后过期

```
expireat(key,timestamp)
```
## pttl
查看键的过期时间还有多少毫秒 **redis2.6及以上可用**

```
pttl(key)
```
## pexpire
设置键在指定毫秒之后过期**redis2.6及以上可用**

```
pexpire(key,milliseconds)
```
## pexpireat
将一个毫秒级精度的unix时间梭设置为给定的键的过期时间**redis2.6及以上可用**

```
pexpireat(key,timestamp-milliseconds)
```

