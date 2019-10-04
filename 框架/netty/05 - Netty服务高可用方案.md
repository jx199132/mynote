# 说明
1 项目采用Springboot + Netty + Websocket实现聊天功能（支持单发，群发）

2 高可用 采用 Nginx + keepalived 


# 项目代码
## pom文件

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>

<dependency>
	<groupId>io.netty</groupId>
	<artifactId>netty-all</artifactId>
</dependency>

<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20180813</version>
</dependency>
```

## 配置文件

```
netty.server.port=7777

server.port=8081
```

## 代码

### 启动类

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ImApplication {

	public static void main(String[] args) {
		SpringApplication.run(ImApplication.class, args);
	}
}

```

### Netty服务类

```
import javax.annotation.PostConstruct;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;

import com.hp.im.netty.handler.HttpRequestHandler;
import com.hp.im.netty.handler.IMTextChannelHandler;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.group.ChannelGroup;
import io.netty.channel.group.DefaultChannelGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.codec.http.websocketx.WebSocketServerProtocolHandler;
import io.netty.handler.stream.ChunkedWriteHandler;
import io.netty.util.concurrent.ImmediateEventExecutor;

@Component
@Configuration
@ConfigurationProperties(prefix = "netty.server")
public class IMServer {
	
	private int port;
	
	private final ChannelGroup channelGroup = new DefaultChannelGroup(ImmediateEventExecutor.INSTANCE);
	
	public void setPort(int port) {
		this.port = port;
	}

	@PostConstruct
	private void init() {
		//需要开启一个新的线程来执行netty server 服务器
        new Thread(new Runnable() {
            public void run() {
            	start();
            }
        }).start();
	}
	
	private void start() {
		//服务端需要2个线程组  boss处理客户端连接  work进行客服端连接之后的处理
		EventLoopGroup boss = null;
		EventLoopGroup work = null;
		try {
			boss = new NioEventLoopGroup();
			work = new NioEventLoopGroup();
			
			ServerBootstrap bootstrap = new ServerBootstrap();
			
			//服务器 配置
			bootstrap.group(boss,work);
			bootstrap.channel(NioServerSocketChannel.class);
			bootstrap.childHandler(new ChannelInitializer<SocketChannel>() {
				@Override
				protected void initChannel(SocketChannel socketChannel) throws Exception {
					// HttpServerCodec：将请求和应答消息解码为HTTP消息
			        socketChannel.pipeline().addLast("http-codec",new HttpServerCodec());
			        // ChunkedWriteHandler：向客户端发送HTML5文件
			        socketChannel.pipeline().addLast("http-chunked",new ChunkedWriteHandler());
			        // HttpObjectAggregator：将HTTP消息的多个部分合成一条完整的HTTP消息
			        socketChannel.pipeline().addLast("aggregator",new HttpObjectAggregator(64 * 1024));
			        
			        socketChannel.pipeline().addLast(new HttpRequestHandler("/ws", channelGroup));
			        
			        //用于处理websocket, /ws为访问websocket时的uri
			        socketChannel.pipeline().addLast("webSocketServerProtocolHandler", new WebSocketServerProtocolHandler("/ws", true));
			        // 进行设置心跳检测
			        //socketChannel.pipeline().addLast( new IdleStateHandler(readerIdleTime, writerIdleTime, allIdleTime, TimeUnit.SECONDS) );
			        
			        // 配置通道处理  来进行业务处理
			        socketChannel.pipeline().addLast(new IMTextChannelHandler(channelGroup));
			        
				}
			});
			bootstrap.option(ChannelOption.SO_BACKLOG,1024).childOption(ChannelOption.SO_KEEPALIVE,true);
			//绑定端口  开启事件驱动
			System.out.println("【服务器启动成功========端口："+port+"】");
			Channel channel = bootstrap.bind(port).sync().channel();
			channel.closeFuture().sync();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}finally {
			//关闭资源
            boss.shutdownGracefully();
            work.shutdownGracefully();
		}
        
	}
	
}

```

### RequestHandler

```
import java.util.LinkedList;
import java.util.List;

import org.json.JSONObject;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.group.ChannelGroup;
import io.netty.handler.codec.http.FullHttpRequest;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
import io.netty.util.AttributeKey;

public class HttpRequestHandler extends SimpleChannelInboundHandler<FullHttpRequest> { //1
    private final String wsUri;
    private final ChannelGroup group;
    
    private boolean isConn = false;
    
    public HttpRequestHandler(String wsUri, ChannelGroup group) {
    	this.wsUri = wsUri;
    	this.group = group;
	}

	@Override
    public void channelRead0(ChannelHandlerContext ctx, FullHttpRequest request) throws Exception {
    	String uri = request.uri();
    	if(uri != null && uri.startsWith(wsUri)) {
    		String id = uri.replace("/ws/", "");
    		ctx.channel().attr(AttributeKey.valueOf("id")).set(id);
    		ctx.fireChannelRead(request.retain());
    	}
    }
    
	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		if(isConn == true) {
			String id = (String) ctx.channel().attr(AttributeKey.valueOf("id")).get();
			group.add(ctx.channel());
			System.out.println("新用户加入，ChannelId ：" + id + " , 在线人数 ：" + group.size());
			
			JSONObject json = new JSONObject();
	    	json.put("welcome", "你好：" + id + " , 欢迎连接!" + " , hashcode : " + this.hashCode());
	    	json.put("onlineNum", group.size());
	    	List<String> list = new LinkedList<>();
	    	group.forEach(c -> {
	    		list.add((String) c.attr(AttributeKey.valueOf("id")).get());
	    	});
	    	json.put("onlines", list);
	    	
	    	JSONObject map = new JSONObject();
	    	map.put("conn", json);
	    	ctx.channel().writeAndFlush(new TextWebSocketFrame(map.toString()));
	    	
	    	group.stream().filter(c -> {
	    		String currentId = (String) c.attr(AttributeKey.valueOf("id")).get();
	    		return !currentId.equals(id);
	    	}).forEach(c -> {
	    		json.put("welcome", "欢迎" + id + "加入!");
	    		c.writeAndFlush(new TextWebSocketFrame(map.toString()));
	    	});
	    	isConn = false;
		}
	}

	@Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
    	isConn = true;
    }
	
	@Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
		String id = (String) ctx.channel().attr(AttributeKey.valueOf("id")).get();
        System.out.println("用户下线: " + id);
        group.remove(ctx.channel());
        
        JSONObject json = new JSONObject();
        List<String> list = new LinkedList<>();
    	group.forEach(c -> {
    		list.add((String) c.attr(AttributeKey.valueOf("id")).get());
    	});
    	json.put("onlines", list);
		json.put("用户离线", id);
		json.put("onlineNum", group.size());
        
        group.stream().filter(c -> {
    		String currentId = (String) c.attr(AttributeKey.valueOf("id")).get();
    		return !currentId.equals(id);
    	}).forEach(c -> {
    		JSONObject map = new JSONObject();
	    	map.put("conn", json);
	    	c.writeAndFlush(new TextWebSocketFrame(map.toString()));
    	});
        
    }
}
```


### 文字处理Handler

```
import java.time.LocalDateTime;

import org.json.JSONObject;

import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.group.ChannelGroup;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
import io.netty.util.AttributeKey;

public class IMTextChannelHandler extends SimpleChannelInboundHandler<TextWebSocketFrame>{
	
	private final ChannelGroup group;
	
	public IMTextChannelHandler(ChannelGroup channelGroup) {
		this.group = channelGroup;
	}
	
	@Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {
        Channel channel = ctx.channel();
        String id = (String)channel.attr(AttributeKey.valueOf("id")).get();
        System.out.println(channel.remoteAddress() + ": " + msg.text());
        JSONObject json = new JSONObject(msg.text());
        if(json.has("sendToId")) {
        	group.stream().filter(c -> {
        		String toId = (String) c.attr(AttributeKey.valueOf("id")).get();
        		return toId.equals(json.getString("sendToId"));
        	}).findFirst().get().writeAndFlush(new TextWebSocketFrame("来自: " + json.getString("sendToId") + " ， data is : " + json.getString("data")));
        }else {
        	ctx.channel().writeAndFlush(new TextWebSocketFrame("来自服务端: " + LocalDateTime.now() + " , 你好 ： " + id + " , data is : " + json.getString("data")));
        }
    }
    
	
	
    @Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
    	ctx.channel().writeAndFlush(new TextWebSocketFrame("你好：" + ctx.channel().attr(AttributeKey.valueOf("id")).get()));
		super.channelActive(ctx);
	}
}

```

### 前端测试页面

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>Socket</title>
        <script type="text/javascript">
            var websocket;

            //如果浏览器支持WebSocket
            if(window.WebSocket){


            }else{
                alert("浏览器不支持WebSocket");
            }

            function conn() {
                var id = document.getElementById("id").value;
                websocket = new WebSocket("ws://localhost:7878/ws" + "/" + id);  //获得WebSocket对象

                //当有消息过来的时候触发
                websocket.onmessage = function(event){
                    var respMessage = document.getElementById("respMessage");
                    respMessage.value = respMessage.value + "\n" + event.data;
                }

                //连接关闭的时候触发
                websocket.onclose = function(event){
                    var respMessage = document.getElementById("respMessage");
                    respMessage.value = respMessage.value + "\n断开连接";
                }

                //连接打开的时候触发
                websocket.onopen = function(event){
                    var respMessage = document.getElementById("respMessage");
                    respMessage.value = "建立连接";
                }
            }


            function sendMsg(msg) { //发送消息
                if(window.WebSocket){
                    if(websocket.readyState == WebSocket.OPEN) { //如果WebSocket是打开状态
                        var sendId = document.getElementById("sendToId").value;
                        var json = {};//定义一个Json对象
                        if(sendId != null && sendId != "" && sendId.length > 0){
                            json.sendToId = sendId;
                        }
                        json.data = msg;
                        //将json对象转成字符串发送
                        websocket.send(JSON.stringify(json)); //send()发送消息
                    }
                }else{
                    return;
                }
            }
        </script>
    </head>
<body>
    <form onsubmit="return false">
		<h3>你的ID</h3>
		<input id = "id"/>	<input onclick="conn()" type="button" value="建立连接"/>
		<br/>
		<h3>发送给谁（ID）</h3>
		<input id = "sendToId"/>
		<br/>
		<h3>发送消息</h3>
        <textarea style="width: 300px; height: 200px;" name="message"></textarea>
        <input type="button" onclick="sendMsg(this.form.message.value)" value="发送"><br>
        <h3>服务端信息</h3>
        <textarea style="width: 300px; height: 200px;" id="respMessage"></textarea>
        <input type="button" value="清空" onclick="javascript:document.getElementById('respMessage').value = ''">
    </form>
</body>
</html>
```


git地址 https://gitee.com/jiuxiao/im.git




# 高可用说明

1 利用keepalived + Nginx 完成高可用


2 Nginx配置支持WebSocket

将下面代码放到Nginx配置文件的HTTP块下

```
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream websocket {
    server localhost:7777;
}

server {
    listen 81;
    location ^~ /ws {
        proxy_pass http://websocket;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }

}

```

3 客户端访问地址改成Keepalived指向的虚拟IP
