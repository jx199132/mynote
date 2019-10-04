# 编写TCP服务端
Vert.x 可以轻松地编写非阻塞的 TCP 客户端和服务器
## 创建TCP服务端

```
NetServer server = vertx.createNetServer();
```
## 配置 TCP 服务器
如果你不想默认值，可以将服务器配置通过传入一个NetServerOptions实例来创建它
```
NetServerOptions options = new NetServerOptions().setPort(4321);
NetServer server = vertx.createNetServer(options);
```
## 启动服务器监听
使用listen告诉服务监听传入的请求

```
NetServer server = vertx.createNetServer();
server.listen(1234, "localhost");
```
不指定的情况下，地址默认使用0.0.0.0，端口默认下为0，如果为的0话，表示随机使用一个没有被占用的端口

在listen方法后面设置一个handler参数，可以在监听成功后，执行handler

```
NetServer server = vertx.createNetServer();
server.listen(1234, "localhost", res -> {
  if (res.succeeded()) {
    System.out.println("Server is now listening!");
  } else {
    System.out.println("Failed to bind!");
  }
});
```

## 连接通知
当有客户端连接进来时，如果希望得到通知的话，可以如下

```
NetServer server = vertx.createNetServer();
//设置客户端连接的handler
server.connectHandler(socket -> {
  // Handle the connection in here
});
```

## 从Socket读取数据
从socket读取数据要在socket上设置handler。

每次在socket上接收到数据Buffer实例，将调用此处理程序

```
NetServer server = vertx.createNetServer();
//设置客户端连接的handler
server.connectHandler(socket -> {
  //设置读取数据的handler
  socket.handler(buffer -> {
    System.out.println("I received some bytes: " + buffer.length());
  });
});
```

## 数据写入socket
使用write写到socket
```
Buffer buffer = Buffer.buffer();
buffer.appendFloat(12.34f).appendInt(123);
socket.write(buffer);

// Write a string in UTF-8 encoding
socket.write("some data");

// Write a string using the specified encoding
socket.write("some data", "UTF-16");
```
写操作是异步的，调用返回之后可能不会发生

## 关闭Handler
如果你想要关闭socket时得到通知，可以设置closeHandler

```
socket.closeHandler(v -> {
  System.out.println("The socket has been closed");
});
```
## 处理异常
socket出现异常时，可以使用exceptionHandler方法，注册一个handler来监听这个事件

```
server.connectHandler(socket -> {
	socket.exceptionHandler(ex -> {
		//do exception
	});
});
```
## Event bus write handler
每个套接字都会自动在事件总线上注册一个处理程序，当在这个处理程序中接收到缓冲区时，它将会把数据写入到这个缓冲区。

这使您可以通过将缓冲区发送到该处理程序的地址，将数据写入可能位于完全不同Verticle或甚至不同Vert.x实例中的套接字

这个handler的地址可以用writeHandlerID方法获得

## 本地和远程地址
NetSocket 的本地地址可以通过 localAddress 来检索。

NetSocket 的远程地址可以通过 romoteAddress 来检索(即连接的另一端的地址)

## 从类路径中发送文件或资源
sendFile方法可以将文件或类路径里的资源直接写入socket。因为其可以借助操作系统内核支持的操作来完成，所以这是一种很有效的发送文件的方式

https://vertx.io/docs/vertx-core/java/#classpath

## 流式套接字
NetSocket的实例也是ReadStream 和 WriteStream 的实例，所以它们可以用来向其他读写流传输数据

获取更多信息请参见 https://vertx.io/docs/vertx-core/java/#streams

## 升级至SSL/TLS连接
一个非SSL/TLS连接可以使用 upgradeToSsl 升级至SSL/TLS。

服务端和客户端必须配置SSL/TLS才能正常工作。详情请参考 https://vertx.io/docs/vertx-core/java/#ssl 章节。

## 关闭TCP服务器
调用 close 方法可以关闭服务器，关闭服务器意味着关闭任何打开的连接和释放所有的服务器资源

关闭实际上是异步处理的，并且可能在调用之后一段时间才能完成。如果你想要在真正关闭连接的时候接收到通知，你可以传递一个Handler

这个处理器将会在完全关闭完成的时候被调用

```
server.close(res -> {
   if (res.succeeded()) {
       System.out.println("Server is now closed");
   } else {
       System.out.println("close failed");
   }
});
```

## 自动清理verticles
如果您从内部的verticles创建TCP服务器和客户端，当verticles被卸载的时候会自动将这些服务器和客户端进行关闭

## 扩展-共享TCP服务器
任何TCP服务器的处理程序总是在相同的事件循环线程上执行的。

这意味着您在多核心的处理器只部署了一个服务器实例，那么您的服务器上最多只能使用一个内核。

为了合理使用多内核处理器，您将会需要在服务器上部署更多的实例。

您可以在代码中以编程的方式实例化更多的实例

```
for (int i = 0; i < 10; i++) {
    NetServer server = vertx.createNetServer();
    server.connectHandler(socket -> {
       socket.handler(buffer -> {
          // Just echo back the data
          socket.write(buffer);
       });
    });
}
```



或者，如果您正在使用verticles，则可以通过命令行的方式上使用 instances 选项来简单地部署多个服务器Verticles实例

```
vertx run com.mycompany.MyVerticle -instances 10
```


或者以编程的方式部署你的实例:

```
DeploymentOptions options = new DeploymentOptions().setInstance(10);
vertx.deployVerticle("com.mycompany.MyVerticle", options);
```

这时候，你也许会问：“一台主机的确定端口，怎么能有多个服务器监听呢？部署多个实例难道不会造成端口冲突吗？”

Vert.x在这里耍了点小花招。*

当你在同一个端口部署另一个服务器时，其实并没有真的创建一个新服务器监听这个端口。

相反，内部其实只维护了一个服务器，但是链接传入时，会用轮询的方式将链接分发给任意的connect handler。

因此，Vert.x的TCP 服务器可以在单线程的情况下，扩展到多个可用的cpu核心。


# 编写TCP客户端
## 创建TCP 客户端

```
NetClient client = vertx.createNetClient()
```
## 配置TCP 客户端

```
NetClientOptions options = new NetClientOptions().setConnectTimeout(10000);
NetClient client = vertx.createNetClient(options);
```
## 创建链接
指定了服务器的port和host后，可以使用connect方法创建到服务器的链接。之后handler将被调用，当链接成功创建，传入的参数会包含一个NetSocket；如果创建失败，传入的参数将包含失败对象

```
NetClientOptions options = new NetClientOptions().setConnectTimeout(10000);
NetClient client = vertx.createNetClient(options);
client.connect(4321, "localhost", res -> {
  if (res.succeeded()) {
    System.out.println("Connected!");
    NetSocket socket = res.result();
  } else {
    System.out.println("Failed to connect: " + res.cause().getMessage());
  }
});
```
## 配置尝试连接的次数
客户端可以被配置成链接失败时自动重连。有两个方法，setReconnectInterval和setReconnectAttempts

当前Vert.x不会尝试重连，这两个特性仅仅在链接创建时可用


```
NetClientOptions options = new NetClientOptions().
    setReconnectAttempts(10).
    setReconnectInterval(500);

NetClient client = vertx.createNetClient(options);
```

## 配置服务器和客户端使用 SSL/TLS
TCP客户端/服务器通过配置可以使用Transport Layer Security(前身是大名鼎鼎的SSL)。

是否使用SSL/TLS 对API没有影响，在NetClientOptions和NetServerOptions实例上配置。

## 在服务端启用 SSL/TLS
通过设置ssl可以启用 SSL/TLS

