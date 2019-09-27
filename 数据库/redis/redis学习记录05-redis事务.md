# 准备
## 悲观锁
悲观锁(Pessimistic Lock), 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。
## 乐观锁
乐观锁(Optimistic Lock), 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库如果提供类似于write_condition机制的其实都是提供的乐观锁。
# 说明
redis使用的就是乐观锁，来提高吞吐量
关系型数据库使用的就是悲观锁

# redis中事务命令
## multi
标记一个事务块的开始
## exec
执行所有事务块内的命令
## watch
监视给定的key,当exec时候如果监视的key从调用watch后发生过变化，则整个事务会失败。
## unwatch
取消 WATCH 命令对所有 key 的监视
## discard
取消事务，放弃执行事务块内的所有命令
# 示例

```
private static void tx() throws InterruptedException{
		
		Jedis jedis = new Jedis("192.168.95.128", 6379);
		String key = "txkey";  
        jedis.watch(key);  
        Transaction tx = jedis.multi();  
        tx.incr(key);
        Thread.sleep(2000);
        tx.incr(key);  
        Thread.sleep(2000);
        tx.incr(key);  
        Thread.sleep(2000);
        List<Object> result = tx.exec();  
        if(result == null || result.isEmpty()){  
            System.out.println("Transaction error...");//可能是watch-key被外部修改，或者是数据操作被驳回  
            return;  
        }  
        for(Object rt : result){  
            System.out.println(rt.toString());  
        } 
	}
```

### 无干扰直接执行此方法控制台将正常输出结果
### 人为的干扰那么事务将无法正常提交
干扰方式：
 第一步正常执行此方法
 第二步在此方法没有执行完毕 ，sleep期间，通过一些手段人为的修改值，那么事务将会回滚
 例如 ： 直接连接redis机器，通过redis-cli -h host -p 连接上redis之后执行命令
 192.168.95.128:6379> incr txkey 来干扰。


