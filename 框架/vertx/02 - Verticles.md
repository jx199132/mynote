# 编写Verticles
所有编写的Verticle类都必须继承Verticle接口。为了方便，一般情况下我们可以继承AbstractVerticle
```
public class MyVerticle extends AbstractVerticle {

  // Called when verticle is deployed
  public void start() {
  }

  // Optional - called when verticle is undeployed
  public void stop() {
  }

}
```
当verticle部署运行时，会调用start方法；当start方法运行完后，就认为verticle已经启动。当verticle卸载时，会调用stop方法；当stop方法运行完成后，就认为verticle已经卸载

# 异步Verticles启动与关闭(Asynchronous Verticle start and stop)
如果你需要在verticle的start方法中执行比较耗费时间的操作时，你不能为了等待其部署，而去阻塞。这样就违背了黄金原则。那么就应该使用异步的操作

```
public class MyVerticle extends AbstractVerticle {

  public void start(Future<Void> startFuture) {
    //do something
  }
}
```
# Verticle的分类
- Standard Verticles 基本的类型。他们总是使用event loop线程执行。
- Worker Verticles 他们通过work pool里面的线程来运行。一个实例不能被多于一个线程使用。
- Multi-threaded worker verticles 他们通过work pool里面的线程来运行。他们的一个实例可以被多个线程并行的执行。

## Standard verticles
标准verticles当创建和调用start方法时分配一个event loop。调用执行都在相同event loop上。

这意味着我们可以保证您的verticles实例中的所有代码总是都执行相同的事件循环上 (只要你不调用它自己创建的线程!)

这意味着可以在程序里作为单线程编写所有的代码，把担心线程和扩展的问题交给Vert.x。没有更多令人担忧的同步和更多不稳定的问题，也避免了多线程死锁的问题。

## Worker verticles
Worker verticles就像标准的verticles一样，但不使用事件循环执行，从 Vert.x worker线程池使用一个线程（Background Workers）。

worker verticles 专为调用阻塞的代码，因为他们不会阻止任何事件循环，Worker verticle实例永远不会有多个线程并行执行

## Multi-threaded worker verticles
和worker verticle很像，但是他可以被并行的执行(能被多个线程同时运行)


# 以编程方式部署 verticles（还可以通过vertx的工具箱形式部署，具体查看部署文档）

```
Vertx vertx = Vertx.vertx(new VertxOptions().setWorkerPoolSize(40));
Verticle myVerticle = new MyVerticle();
vertx.deployVerticle(new MyVerticle());
```

# 等待verticle部署完成（Waiting verticle for deployment to complete
verticle的部署是异步的。如果你希望在verticle部署完成时得到通知，你可以注册一个handler来实现：

```
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", res -> {
  if (res.succeeded()) {
    System.out.println("Deployment id is: " + res.result());
  } else {
    System.out.println("Deployment failed!");
  }
});
```
部署成功后会返回deployID，deployID在卸载的时候会用到

# 卸载verticle（Undeploying verticle deployments）
通过deployID来卸载。卸载也是异步的

```
vertx.undeploy(deploymentID, res -> {
  if (res.succeeded()) {
    System.out.println("Undeployed ok");
  } else {
    System.out.println("Undeploy failed!");
  }
});
```

# 指定verticle实例数
部署verticle时，我们可以通过指定数量来决定我们部署多少个相同的verticle

```
DeploymentOptions options = new DeploymentOptions().setInstances(16);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```
用于跨多个内核轻松扩展，服务器是多核的，你想要部署多个实例来充分利用所有核心。


# 配置verticle
通过json来配置：

```
JsonObject config = new JsonObject().put("name", "tim").put("directory", "/blah");
DeploymentOptions options = new DeploymentOptions().setConfig(config);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```

# 在Verticle里访问环境变量
使用 Java API 访问环境变量和系统属性:

```
System.getProperty("prop");
System.getenv("HOME");
```

# Verticle隔离组
有的时候我们希望将不同的或者相同的verticle进行隔离部署，例如相同的verticle，不同的版本

```
DeploymentOptions options = new DeploymentOptions().setIsolationGroup("mygroup");
options.setIsolatedClasses(Arrays.asList("com.mycompany.myverticle.*",
                   "com.mycompany.somepkg.SomeClass", "org.somelibrary.*"));//此处部署的verticle会进行隔离加载
vertx.deployVerticle("com.mycompany.myverticle.VerticleClass", options);//此处的VerticleClass会使用当前的classload进行加载。
```

# 从命令行运行 Verticles
使用 Vert.x ，通常可以直接在 Maven 或 Gradle 项目中添加 Vert.x core 库依赖。还可以直接从命令行运行 Vert.x verticles

做到这一点，你需要下载和安装一个 Vert.x ，并将安装的 bin 目录添加到 PATH 环境变量。还要确保 PATH 有 Java 8 JDK

现在可以通过使用 vertx run 命令运行 verticles。这里有一些例子：

```
# Run a JavaScript verticle
vertx run my_verticle.js
# Run a Ruby verticle
vertx run a_n_other_verticle.rb
# Run a Groovy script verticle, clustered
vertx run FooVerticle.groovy -cluster
# 你甚至可以不编译直接运行Java源代码！
vertx run SomeJavaSourceFile.java
```

# Context对象
可以使用如下代码来获取上下文对象

```
Context context = vertx.getOrCreateContext();
```
获取到上下文对象后，我们就可以在其上运行代码

```
vertx.getOrCreateContext().runOnContext( (v) -> {
  System.out.println("This will be executed asynchronously in the same context");
});
```
当有多个handler运行在同一个context上时，我们就可以通过context来共享数据

```
final Context context = vertx.getOrCreateContext();
context.put("data", "hello");
context.runOnContext((v) -> {
  String hello = context.get("data");
});
```


# 执行定期和延迟的操作
在Vert.x 中执行定期和延迟的操作是非常常见的。

在标准 verticles 中，不能使用thread sleep引入延迟，这样会止事件循环线程。

可以使用 Vert.x 计时器。定时器可以分为一次性的计时器或定期的计时器。我们
一次性的计时器

```
long timerID = vertx.setTimer(1000, id -> {
    System.out.println("And one second later this is printed");
});
System.out.println("First this is printed");
```


一个单次定时器有一定的延迟之后调用一个事件处理程序，以毫秒为单位表示。

```
long timerID = vertx.setPeriodic(1000, id -> {
    System.out.println("And every second this is printed");
});
System.out.println("First this is printed");

# 若要取消则

vertx.cancelTimer(timerID);
```


# Verticles 自动清理
如果在Verticles创建了定时器，那么这些定时器在Verticles卸载的时候自动关闭