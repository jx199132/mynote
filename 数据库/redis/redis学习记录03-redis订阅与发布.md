订阅sub/发布pub的特点是订阅者负责订阅频道，发布者负责向频道发送消息，每当有消息被发送给订阅频道时，所有该频道的订阅者都能够收到消息。

# 命令简介
## subscribe
订阅一个或者多个频道

```
subscribe(channel...)
```
## unsubscribe
退订一个或者多个频道

```
unsubscribe([channel...])如果没有指定任何频道，那么退订全部
```
## publish
给指定频道发布消息

```
publisth(channel,message)
```
## psubscribe
订阅与给定模式相匹配的所有频道

```
psubscribe(pattern...)
```
## punsubscribe
退订指定模式相匹配的所有频道，如果不指定任何模式，那么退订全部

```
punsubscribe([pattern...])
```
# 示例演示(jedis)

## 发布者,读取控制台输入内容进行发布

```
package com.jx.redis;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

public class Publisher {
    private final JedisPool jedisPool;

    public Publisher(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    public void start() {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        Jedis jedis = jedisPool.getResource();
        while (true) {
            String line = null;
            try {
                line = reader.readLine();
                jedis.publish("mychannel", line);
                System.out.println("pub : " + line);
                if ("quit".equals(line)) {
                	 break;
                } else {
                   
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
         }
    }
}
```

## 订阅者收到消息处理类

```
package com.jx.redis;
import redis.clients.jedis.JedisPubSub;


public class Subscriber extends JedisPubSub {
	
	private String name;
	
	public Subscriber(String name) {
		this.name = name;
    }

    public void onMessage(String channel, String message) {
        System.out.println(String.format(name + "接收消息, 频道 %s, message %s", channel, message));
        if("quit".equals(message)){
        	System.out.println(name + "退订");
        	super.unsubscribe();
        }
    }
}
```
## 订阅者线程类

```
package com.jx.redis;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;


public class SubThread extends Thread {
    private final JedisPool jedisPool;
    private Subscriber subscriber;

    private final String channel = "mychannel";

    public SubThread(JedisPool jedisPool , String name) {
        this.jedisPool = jedisPool;
        this.subscriber = new Subscriber(name);
    }

    @Override
    public void run() {
        System.out.println(String.format("订阅频道 %s", channel));
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.subscribe(subscriber, channel);
        } catch (Exception e) {
            System.out.println(String.format("订阅频道error, %s", e));
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }
}
```

## 测试类

```
 public static void main( String[] args )
    {
        // 替换成你的reids地址和端口
        String redisIp = "192.168.95.128";
        int reidsPort = 6379;
        JedisPool jedisPool = new JedisPool(new JedisPoolConfig(), redisIp, reidsPort);
        System.out.println(String.format("启动redis连接池 host %s, port %d", redisIp, reidsPort));

        SubThread subThread = new SubThread(jedisPool,"订阅者A");
        subThread.start();
        
        SubThread subThread2 = new SubThread(jedisPool,"订阅者B");
        subThread2.start();

        Publisher publisher = new Publisher(jedisPool);
        publisher.start();
    }
```

## 控制台输入输出

```
启动redis连接池 host 192.168.95.128, port 6379
订阅频道 mychannel
订阅频道 mychannel
11
pub : 11
订阅者B接收消息, 频道 mychannel, message 11
订阅者A接收消息, 频道 mychannel, message 11
11
pub : 11
订阅者A接收消息, 频道 mychannel, message 11
订阅者B接收消息, 频道 mychannel, message 11
ff
pub : ff
订阅者A接收消息, 频道 mychannel, message ff
订阅者B接收消息, 频道 mychannel, message ff
quit
pub : quit
订阅者A接收消息, 频道 mychannel, message quit
订阅者A退订
订阅者B接收消息, 频道 mychannel, message quit
订阅者B退订
```