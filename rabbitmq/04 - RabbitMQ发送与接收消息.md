# 详解
## 发送消息

### 示例

基本发送：四个参数（交换器名称，路由键，消息类型，消息内容）
```
byte[] messageBodyBytes = "Hello , world! ". getBytes();
channel.basicPublish(exchangeName , routingKey , null , messageBodyBytes);
```

自定义的发送方式：

- 投递模式（deliveryMode = 2）

- contentType = 文本

- 优先级priority（1）


```
BasicProperties prop = new AMQP.BasicProperties().builder()
					.contentType("text/p1ain")
					.deliveryMode(2)
					.priority (1).userId("hidden").build();
			
channel.basicPublish(EXCHANGE_NAME , "routingKey", prop, message.getBytes());
```
带有Header的方式发送：

```
Map<String, Object> headers = new HashMap<String, Object>() ;
headers.put( "loca1tion" , "here " );
headers.put( " time " , " today" );

BasicProperties prop = new AMQP.BasicProperties().builder()
		.headers(headers)
		.build();

channel.basicPublish(EXCHANGE_NAME , "routingKey", prop, message.getBytes());
```
带有过期时间的发送：

```
BasicProperties prop = new AMQP.BasicProperties().builder()
					.expiration("6000")
					.build();
			
channel.basicPublish(EXCHANGE_NAME , "routingKey", prop, message.getBytes());
```

### 参数说明
- **exchange**: 交换器的名称，指明消息需要发送到哪个交换器中 。如果设置为空字符串，则消息会被发送到 RabbitMQ 默认的交换器中
- **routingKey** : 路由键，交换器根据路由键将消息存储到相应的队列之中
- **props** : 消息的基本属性集
- **byte [] body** : 消息体 ( pay1oad ) ，真正需要发送的消息
- **mandatory** ：当交换器无法根据自身的类型和路由键找到一个符合条件
的队列，为 true 时RabbitMQ 会调用 Basic.Return 命令将消息返回给生产者，为false时则消息直接被丢弃 

- **immediate**：为true时，如果exchange在将消息route到queue(s)时发现对应的queue上没有消费者，那么这条消息不会放入队列中。当与消息routeKey关联的所有queue(一个或多个)都没有消费者时，该消息会通过basic.return方法返还给生产者



概括来说，mandatory标志告诉服务器至少将该消息route到一个队列中，否则将消息返还给生产者；immediate标志告诉服务器如果该消息关联的queue上有消费者，则马上将消息投递给它，如果所有queue都没有消费者，直接把消息返还给生产者，不用将消息入队列等待消费者了




## 接收消息

RabbitMQ 的消费模式分两种 : 推 ( Push )模式和拉 ( Pull )模式。

推模式采用 channel.basicConsume，而拉模式则是调用 channel.basicGet

### 推模式（Push）

示例代码
```
Consumer consumer = new DefaultConsumer(channel) {
				@Override
				public void handleDelivery(String consumerTag, Envelope envelope, BasicProperties properties,
						byte[] body) throws IOException {
					System.out.println("收到消息 : " + new String(body));
				}
			};
			
channel.basicConsume(QUEUE_NAME, consumer);
```

参数说明
- **queue** : 队列的名称
- **autoAck** : 设置是否自动确认
- **consumerTag**: 消费者标签，用来区分多个消费者
- **noLocal** : 设置为true则表示不能将同一个Collection中生产者发送的消息                  传送给这个Connection中的消费者
- **exclusive** : 设置是否排他  https://www.cnblogs.com/alter888/p/8978463.html
- **arguments** : 设置消费者的其他参数
- **callback** : 设置消费者的回调函数。用来处理RabbitMQ推送过来的消息，比如DefaultConsumer ， 使用时需要客户端重写 (override) 其中的方法


### 拉模式（Pull）


```
GetResponse res = channel.basicGet(queue, autoAck);
res.getBody();
```
