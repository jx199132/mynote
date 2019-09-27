# 说明
Redis的分片（Sharding或者Partitioning）技术是指将数据分散到多个Redis实例中的方法，分片之后，每个redis拥有一部分原数据集的子集。在数据量非常大时，这种技术能够将数据量分散到若干主机的redis实例上，进而减轻单台redis实例的压力。分片技术能够以更易扩展的方式使用多台计算机的存储能力（这里主要指内存的存储能力）和计算能力：

1. 从存储能力的角度，分片技术通过使用多台计算机的内存来承担更大量的数据，如果没有分片技术，那么redis的存储能力将受限于单台主机的内存大小。

2. 从计算能力的角度，分片技术通过将计算任务分散到多核或者多台主机中，能够充分利用多核、多台主机的计算能力。

# 分片算法

采用分片之后，数据将被分散到多个redis实例中，对数据的操作也被分散到每个redis实例中，此时单台主机的压力将大大减轻。
分片的部署，即实例中分片算法被放在哪里？是在分片时需要首先考虑的问题，分片部署方式一般分为以下三种：

1. 在客户端做分片；这种方式在客户端确定要连接的redis实例，然后直接访问相应的redis实例；

2. 在代理中做分片；这种方式中，客户端并不直接访问redis实例，它也不知道自己要访问的具体是哪个redis实例，而是由代理转发请求和结果；其工作过程为：客户端先将请求发送给代理，代理通过分片算法确定要访问的是哪个redis实例，然后将请求发送给相应的redis实例，redis实例将结果返回给代理，代理最后将结果返回给客户端。

3. 在redis服务器端做分片；这种方式被称为“查询路由”，在这种方式中客户端随机选择一个redis实例发送请求，如果所请求的内容不再当前redis实例中它会负责将请求转交给正确的redis实例，也有的实现中，redis实例不会转发请求，而是将正确redis的信息发给客户端，由客户端再去向正确的redis实例发送请求。

# 分片的缺点
上面主要描述了分片的优点，当然分片的存在也有缺陷，例如：

1. 通常无法支持涉及多键的操作；在redis中有很多一次操作多个key的操作，例如求集合交集的SINTER操作，该操作将涉及到多个键，而这多个键有可能被分片到不同的redis实例中，此时就无法执行这种操作。

2. Redis的事务操作中涉及多个键时也不能用；

3. 分片将导致数据处理更加复杂；例如在分片过程中，随着redis实例的增加，数据备份等操作都将会变得更加复杂。

4. Redis目前不支持动态分片操作，扩容和缩容操作都会比较复杂，尤其分片操作部署在客户端时，需要重新配置和启动客户端。在使用过程中缩容用的不多，扩容可以采用后面介绍的预分片策略来缓解此问题。

# Jedis分片连接池
Jedis通过一致性哈希分片算法来实现数据分片，一致性哈希算法：http://blog.csdn.net/cywosp/article/details/23397179/

简单用Jedis分片连接池来实现多Redis的master节点进行数据分片，步骤如下：
1. 部署2个Redis master节点。
2. 用ShardedJedisPool连接Redis maser节点，并将数据根据一致性哈希算法存放到不同的master节点，从而达成数据分片。
客户端代码如下：

```
public class RedisShardPoolTest {  
    static ShardedJedisPool pool;  
    static{  
        JedisPoolConfig config =new JedisPoolConfig();//Jedis池配置  
        config.setMaxActive(500);//最大活动的对象个数  
          config.setMaxIdle(1000 * 60);//对象最大空闲时间  
          config.setMaxWait(1000 * 10);//获取对象时最大等待时间  
          config.setTestOnBorrow(true);  
        String hostA = "10.7.12.52";  
          int portA = 6379;  
          String hostB = "10.7.112.52";  
          int portB = 6379;  
        List<JedisShardInfo> jdsInfoList =new ArrayList<JedisShardInfo>(2);  
        JedisShardInfo infoA = new JedisShardInfo(hostA, portA);  
        //infoA.setPassword("redis.360buy");  
        JedisShardInfo infoB = new JedisShardInfo(hostB, portB);  
        //infoB.setPassword("redis.360buy");  
        jdsInfoList.add(infoA);  
        jdsInfoList.add(infoB);  
         
        pool =new ShardedJedisPool(config, jdsInfoList, Hashing.MURMUR_HASH,  
Sharded.DEFAULT_KEY_TAG_PATTERN);  
    }  
     
    /** 
     * @param args 
     */  
    public static void main(String[] args) {  
        for(int i=0; i<100; i++){  
           String key =generateKey();  
           //key += "{aaa}";  
           ShardedJedis jds =null;  
           try {  
               jds =pool.getResource();  
               System.out.println(key+":"+jds.getShard(key).getClient().getHost());  
               System.out.println(jds.set(key,"1111111111111111111111111111111"));  
           }catch (Exception e) {  
               e.printStackTrace();  
           }  
           finally{  
               pool.returnResource(jds);  
           }  
        }  
    }  
   
    private static int index = 1;  
    public static String generateKey(){  
        return String.valueOf(Thread.currentThread().getId())+"_"+(index++);  
    }  
}  
```
通过执行结果会发现这100条数据会被分开存放到10.7.12.52和10.7.112.52机器上的master节点中，查看源码会发现jds.set(key,value)方法会调用jds.getShared(key)来根据key的hash值来获取该key应该存放到那个节点上。