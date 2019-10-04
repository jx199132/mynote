# 初始化一个Vertx

```
Vertx vertx = Vertx.vertx( new VertxOptions().setWorkerPoolSize(40) );
```

# 事件模型流程（Don’t call us, we’ll call you ）
Vert.x Api 是很大程度上由事件驱动的。当事情发生在Vert.x会通过回调方式向您发送events。

一些示例events:

- 计时器激活
- socket收到数据
- 从磁盘读取数据
- 发生了异常
- HTTP 服务器收到请求


# 不要组赛我（Don’t block me!）

除了极少数例外 (一些文件系统操作的“同步”结束)，没有一个 Vert.x Api 阻塞调用线程


如果可以立即提供的结果，它将立即返回，你通常会提供一个handle来接收过一段时间的事件

由于Vert.x API没有任何阻塞的线程，这意味着你可以使用Vert.x来处理只是使用小数目线程的大量并发

常规阻塞API使用线程可能会阻塞：
- 从socket读取数据
- 向磁盘写入数据
- 向收件人发送一条消息，等待答复

当线程正在等待结果时它不能做别的-这是实际上是浪费。这意味着，如果你需要大量的并发使用阻塞 APIs，然后你需要大量的线程，以防止您的应用程序停止工作。

线程在他们所需要的内存（例如栈） 和上下文切换方面有开销


# 反应器和多反应器（Reactor and Multi-Reactor）
Vert.x API是事件驱动，Vertx要求使用一种称为event loop线程的处理程序。

因为没有阻塞，event loop可以在短时间内提供大量的事件。例如一个单一的event loop可以非常迅速地处理成千上万的 HTTP 请求。

这个叫做反应器模式（Reactor Pattern）.

Node.js 就是实现此模式。标准的Reactor所有事件都运行在单一事件循环线程。
单个线程的麻烦是在任何一个时间它只能运行在单一的核心上(例如 Node.js 应用

而Vert.x 不同。不是单事件循环，每个 Vertx 实例都维护若干个事件循环。默认情况下，Vert.x选择数量基于在机器上可用的内核数，但可以自己设置。

# 黄金法则 — 不要阻塞事件循环(The Golden Rule - Don’t Block the Event Loop

Vert.x Api 是非阻塞，并且不会堵塞事件循环。 如果你堵塞事件循环,那事件
循环将不能做别的事,因为它被阻塞了。如果所有的事件循环被阻塞了，应用程序将完全停止！

阻塞的例子包括:
- Thread.sleep()
- 等待锁
- 等待互斥体或监视器 (例如同步段)
- 做一个长时间的数据库操作和等待返回
- 做复杂的计算，需要很长的时间。
- 死循环。


Vert.x会自动记录警告。如果你在日志中看到这样的警告，那么你就应该去检查应用

```
Thread vertx-eventloop-thread-3 has been blocked for 20458 ms
```

Vert.x 还将提供 堆栈跟踪 来确定阻塞发生的位置。

如果你想关闭这些警告或更改设置，你可以在创建Vertx对象之前，使用VertxOptions配置


# 运行阻塞代码(Running blocking code
在理想情况下所有Api将使用异步写，但是，现实世界并不是这样。

事实是，大多数库，特别是在JVM的生态,有许多是同步API，许多的方法有可能阻塞

不管如何努力尝试，Vert.x 不能撒上魔法使之同步。

如前所述，直接在事件循环里调用阻塞操作，会妨碍它做任何其他有用的工作。

Vert.x通过调用 executeBlocking指定要执行的阻塞的代码和在执行阻塞的代码时调用返回异步结果处理程序

执行阻塞代码，当阻塞代码执行完成后通过异步回调的方式返回

```
vertx.executeBlocking(future -> {
  // Call some blocking API that takes a significant amount of time to return
  String result = someAPI.blockingMethod("hello");
  future.complete(result);
}, res -> {
  System.out.println("The result is: " + res.result());
});

```


默认情况下，如果 executeBlocking 从相同的上下文 (例如同一垂直实例) 调用几次不同的executeBlocking 则以串行方式执行 。

默认情况下，如果 executeBlocking 在同一环境（例如同一个verticle实例） 多次调用，那么不同的 executeBlocking 将串行执行

如果你不关心这些executeBlocking执行的顺序，那么你可以设置executeBlocking的参数ordered为false。这样的话所有的executeBlocking代码就会并行的在工作线程中执行。 


# 异步协调（Async coordination）
通过Vert.x 的futures类来实现异步返回结果的协调问题。


```
Future<HttpServer> httpServerFuture = Future.future();
httpServer.listen(httpServerFuture.completer());

Future<NetServer> netServerFuture = Future.future();
netServer.listen(netServerFuture.completer());

CompositeFuture.all(httpServerFuture, netServerFuture).setHandler(ar -> {
  if (ar.succeeded()) {
    // All server started  全部成功
  } else {
    // At least one server failed   至少一个失败
  }
});

```

CompositeFuture.all可以将所有的future包含起来，然后判断所有的这些future是否全部成功还是有失败的。如上面的例子


```
Future<String> future1 = Future.future();
Future<String> future2 = Future.future();
CompositeFuture.any(future1, future2).setHandler(ar -> {
  if (ar.succeeded()) {
    // At least one is succeeded  至少有一个是成功的
  } else {
    // All failed  全部失败
  }
});
```
CompositeFuture.any 跟all的意思一样，但是判断成功失败的条件不一样

```
FileSystem fs = vertx.fileSystem();

Future<Void> fut1 = Future.future();
Future<Void> fut2 = Future.future();

fs.createFile("/foo", fut1.completer());
fut1.compose(v -> {
  fs.writeFile("/foo", Buffer.buffer(), fut2.completer());
}, fut2);
fut2.compose(v -> {
  fs.move("/foo", "/bar", startFuture.completer());
}, startFuture);
```
以上代码表明，compose方法可以链式的调用Future