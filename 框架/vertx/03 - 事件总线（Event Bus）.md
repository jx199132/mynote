# 简述
event bus是Vert.x 的中枢神经系统 。

通过Vert.x实例使用eventBus方法得到单一的event bus实例。

事件总线允许您的应用程序相互沟通，不论何种语言，他们写的以及他们是否在同一个 Vert.x 实例，或在一个不同的Vert.x 实例的不同部分。

它甚至允许客户端 JavaScript 运行在浏览器上相同的事件总线进行通信。

事件总线构成了一个分布式对等消息传递系统跨越多个服务器节点和多个浏览器。

事件总线支持发布/订阅，点到点和请求-响应消息。

事件总线 API 是非常简单的。它基本上涉及注册处理程序，注销处理程序和发送和发布消息

# 原理
## 地址（Addressing）
消息通过事件总线发送给一个地址

在Vertx中并没有复杂的地址设计，一个字符串就是一个地址。然而它是明智地使用某种类型，例如使用句点划定一个命名空间

例如：europe.news.feed1, acme.games.pacman, sausages, and X

## 处理者（Handlers）
消息的接受是需要用Handler的，需要在一个地址上注册一个Handler

不同的handler可以注册在同一个地址上

一个handler可以注册在不同的地址上

## 发布和订阅消息（Publish / subscribe messaging）
事件总线支持发布消息

消息发送到某一个地址上，那么意味着所以该地址上的Handler都可以获取该消息

## 点对点的发送和响应消息（Point to point and Request-Response messaging）

事件总线支持点对点的消息服务

消息被发送到一个地址上，在Vertx中，那么该地址只能有一个Handler注册

如果该地址有多个Handler注册，那么将采取不是很严格的轮询调度算法（不会所有Handler都获取，一次只有一个Handler获取到消息，获取的Handler采用轮询的方式）

使用点到点消息传递，发送消息时，可以指定一个可选的*答复处理程序*。

当消息由收件人收到，并且已被处理时，收件人可以选择决定对消息进行回复。如果他们这样做*答复处理程序*将被调用。

当回到在发件人收到的答复时，它也可以被回复。这可以是重复的

这是一种常见的消息传递模式，称为请求-响应模式

## 最大程度的进行消息传递（Best-effort delivery）
Vertx尽最大努力的投递消息，但是如果消息总线出了问题也会导致消息丢失。

如果你的应用程序在意消息丢失，那么你的程序应该具备幂等性，在消息总线恢复之后进行重新发送。

## 消息类型（Types of messages）
Vert.x 允许任何原始/简单类型、 字符串或buffers将作为消息发送。

Vert.x 推荐以json 格式发送消息

然而你不被迫使用 JSON，如果你不想去。事件总线是非常灵活，也支持通过事件总线发送任意对象

# 事件总线API（Event Bus API）

## 获取一个事件总线

```
EventBus eb = vertx.eventBus();
```
每一个vertx实例都有一个单实例的event bus对象

## 注册Handler
这个最简单的方法来注册一个Handler用consumer

```
EventBus eb = vertx.eventBus();

eb.consumer("news.uk.sport", message -> {
  System.out.println("I have received a message: " + message.body());
});
```

当消息到达Handler之后，Handler将被调用。

如上面的代码，这是注册handler的一个简单的方法，通过consumer方法。调用consumer()后返回的是一个MessageConsumer对象。这个对象经常用在卸载handler或者以Stream的形式使用Handler

当然我们也可以按如上代码使用这个对象。先返回一个没有handler集合的MessageConsumer对象，然后再设置相应的handler，代码如下：

```
EventBus eb = vertx.eventBus();

MessageConsumer<String> consumer = eb.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
});
```

当我们在一个集群化的event bus上面注册一个handler时，所有的节点都完成注册，可能需要花费一定的时间。如果我们希望能够在handler注册完成时得到通知，我们可以注册一个completion handler，代码如下：

```
consumer.completionHandler(res -> {
  if (res.succeeded()) {
    System.out.println("The handler registration has reached all nodes");
  } else {
    System.out.println("Registration failed!");
  }
});
```


## 卸载Handler（Un-registering Handlers）

卸载调用 unregister方法

如果是集群化的event bus上需要监听是否完成的话：

```
consumer.unregister(res -> {
  if (res.succeeded()) {
    System.out.println("The handler un-registration has reached all nodes");
  } else {
    System.out.println("Un-registration failed!");
  }
});
```

## 发布消息（Publishing messages）
所有订阅该地址的Handler都可以收到消息
```
eventBus.publish("news.uk.sport", "Yay! Someone kicked a ball");
```

## 发送消息（Sending messages）
点对点的方式
```
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball");
```

## 设置消息头（Setting headers on messages）

```
DeliveryOptions options = new DeliveryOptions();
options.addHeader("some-header", "some-value");
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball", options);
```

## 消息对象（The Message object）
消息对象就是message实例

body就是我们发送的内容

headers就是我们设置的消息的头部

## 确认消息/发送回复（Acknowledging messages / sending replies）
确认一个消息是否收到

发送者：

```
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball across a patch of grass", ar -> {
  if (ar.succeeded()) {
    System.out.println("Received reply: " + ar.result().body());
  }
});
```

接受者：

```
MessageConsumer<String> consumer = eventBus.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
  message.reply("how interesting!");
});
```

## 发送超时（Sending with timeouts）
发送答复处理消息时你可以在DeliveryOptions中指定超时，默认是30秒

如果在该时间内没有收到答复，答复处理程序将调用发送失败的方法

## 发送失败（Send Failures）
消息发送会失败，答复处理程序将调用发送失败的方法

## 消息编解码（Message Codecs）

event bus可以发送任意的对象，只要我们实现这个对象的codec

```
eventBus.registerCodec(myCodec);

DeliveryOptions options = new DeliveryOptions().setCodecName(myCodec.name());

eventBus.send("orders", new MyPOJO(), options);//通过options设置，只对这一次发送管用
```

如果你发送的消息始终是这个类型的，可以如下：

```
eventBus.registerDefaultCodec(MyPOJO.class, myCodec);

eventBus.send("orders", new MyPOJO());
```
取消这个消息编码，用unregisterCodec方法

## 集群EventBus
Event Bus并不仅仅存在于一个单一的Vert.x实例。通过Vert.x集群实例就可以形成一个单一的，分布式的，Event Bus


## 以编程的方式集群（Clustering programmatically）
如果您要以编程方式创建Vert.x集群，并且通过Vertx获取集群的EventBus

```
VertxOptions options = new VertxOptions();
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
    EventBus eventBus = vertx.eventBus();
    System.out.println("We now have a clustered event bus: " + eventBus);
  } else {
    System.out.println("Failed: " + res.cause());
  }
});
```

## 通过命令的形式集群（Clustering on the command line）
通过命令的方式启动Vertx集群
```
vertx run my-verticle.js -cluster
```

## 自动清理 Verticles（Automatic clean-up in verticles）
卸载verticle时，注册在其里面的handler都会被自动的卸载

