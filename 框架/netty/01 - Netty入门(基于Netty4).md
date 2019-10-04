# 准备工作
引入jar即可
```
<dependency>
	<groupId>io.netty</groupId>
	<artifactId>netty-all</artifactId>
	<version>4.1.31.Final</version>
</dependency>
```


# 服务端

服务引导类
```
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class MyServer {
	
	private static int port = 7878;

    public static void main(String[] args){

        //服务启动类
        ServerBootstrap b = new ServerBootstrap();

        /**
         *	实例化两个事件循环，Boss和Worker
         *  Boss负责监听端口，当有客户端连接了，就将客户端分配给Worker
         *  Worker负责处理客户端的具体读写任务
         */
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //设置事件循环组
            b.group(bossGroup, workerGroup);
            //指定所使用的 NIO传输 Channel类型， 会自动根据类型创建工厂
            b.channel(NioServerSocketChannel.class);
            //设置childHandler，对每一个客户端进行初始化工作
            b.childHandler(new MyServerInitializer());//handler在初始化时就会执行，而childHandler会在客户端成功connect后才执行，这是两者的区别

            // 服务器绑定端口监听
            ChannelFuture f = b.bind(port).sync();
            // 监听服务器关闭监听
            f.channel().closeFuture().sync();

            // 可以简写为
            /* b.bind(portNumber).sync().channel().closeFuture().sync(); */
        }catch (Exception e) {
			e.printStackTrace();
		}finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
	
}


```
每一个客户端连接进行初始化工作类
```
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.DelimiterBasedFrameDecoder;
import io.netty.handler.codec.Delimiters;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

public class MyServerInitializer extends ChannelInitializer<SocketChannel>{

	@Override
	protected void initChannel(SocketChannel ch) throws Exception {
		ChannelPipeline pipeline = ch.pipeline();

        // 以("\n")为结尾分割的 解码器
        pipeline.addLast("framer", new DelimiterBasedFrameDecoder(8192, Delimiters.lineDelimiter()));

        // 字符串解码 和 编码
        pipeline.addLast("decoder", new StringDecoder());
        pipeline.addLast("encoder", new StringEncoder());

        // 自己的逻辑Handler
        pipeline.addLast("handler", new MyServerHandler());
	}

}

```
处理每个客户端的请求工作类
```
import java.net.InetAddress;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class MyServerHandler extends SimpleChannelInboundHandler<String> {

	@Override
	protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
		// 收到消息直接打印输出
		System.out.println(ctx.channel().remoteAddress() + " Say : " + msg);
		// 返回客户端消息 - 我已经接收到了你的消息
		ctx.writeAndFlush("Received your message !\n");
	}

	/**
	 *  	覆盖 channelActive 方法 在channel 被启用的时候触发 (在建立连接的时候)
	 */
	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println("RamoteAddress : " + ctx.channel().remoteAddress() + " active !");
		ctx.writeAndFlush("Welcome to " + InetAddress.getLocalHost().getHostName() + " service!\n");
		super.channelActive(ctx);
	}

	/**
	 * 	 异常处理:
	 *  	1  打印异常并
	 *  	2  关闭该Channel
	 */
	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
		cause.printStackTrace();
		ctx.close();
	}

	@Override
	public void channelInactive(ChannelHandlerContext ctx) throws Exception {
		System.out.println("disconnect : " + ctx.channel().remoteAddress() + " disconnect !");
		ctx.close();
	}
	
	
}

```

# 客户端

客户端启动引导类
```
import java.io.BufferedReader;
import java.io.InputStreamReader;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;

public class MyClient {
    
    public static String host = "127.0.0.1";
    public static int port = 7878;

    public static void main(String[] args) throws InterruptedException{
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group)
            .channel(NioSocketChannel.class)
            .handler(new MyClientInitializer());

            // 连接服务端
            Channel ch = b.connect(host, port).sync().channel();
            // 控制台输入
            BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
            while (true){
                String line = in.readLine();
                if ("end".equals(line)) {
                    break;
                }
                /**
                 * 	向服务端发送在控制台输入的文本 并用"\r\n"结尾
                 * 	之所以用\r\n结尾 是因为我们在handler中添加了 DelimiterBasedFrameDecoder 帧解码。
                 * 	这个解码器是一个根据\n符号位分隔符的解码器。所以每条消息的最后必须加上\n否则无法识别和解码
                 * */
                ch.writeAndFlush(line + "\r\n");
            }
        }catch (Exception e) {
			e.printStackTrace();
		}finally {
			System.out.println("shutdown cient");
            group.shutdownGracefully().sync();
        }
    }
}
```
客户端初始化Handler
```
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.DelimiterBasedFrameDecoder;
import io.netty.handler.codec.Delimiters;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

public class MyClientInitializer extends ChannelInitializer<SocketChannel> {

	@Override
	protected void initChannel(SocketChannel ch) throws Exception {
		 ChannelPipeline pipeline = ch.pipeline();

	        /*
	         * 	这个地方的 必须和服务端对应上。否则无法正常解码和编码
	         * 
	         * 	解码和编码 我将会在下一张为大家详细的讲解。再次暂时不做详细的描述
	         * 
	         * */
	        pipeline.addLast("framer", new DelimiterBasedFrameDecoder(8192, Delimiters.lineDelimiter()));
	        pipeline.addLast("decoder", new StringDecoder());
	        pipeline.addLast("encoder", new StringEncoder());
	        
	        // 客户端的逻辑
	        pipeline.addLast("handler", new MyClientHandler());
	}

}

```
客户端处理与服务端的通信工作类
```
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class MyClientHandler extends SimpleChannelInboundHandler<String> {

	@Override
	protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
		System.out.println("Server say : " + msg);
	}

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println("Client active ");
		super.channelActive(ctx);
	}

	@Override
	public void channelInactive(ChannelHandlerContext ctx) throws Exception{
		System.out.println("Client close ");
		ctx.close();
	}
}

```

