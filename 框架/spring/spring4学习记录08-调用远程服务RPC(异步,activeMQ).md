# 简介
上一篇文章说的是应用之间同步进行交互的场景，这里介绍异步的使用。
# 概念
同步，当客户端调用远程方法时，客户端必须等待远程方法完成之后，才能继续执行，即使远程方法并不返回任何消息，客户端也必须阻塞到服务完成。
异步，客户端不需要等待服务器处理消息，甚至不需要等待客户端把消息发送给了服务端，就可以继续执行后面的流程，这是因为客户端假定服务端最终可以收到并处理这条消息。

# 发送消息
在异步消息中主要有两个概念：消息代理和目的地。
当一个应用发送消息时，会将消息交给一个消息代理，消息代理类似邮局。消息代理可以确保消息被投递到指定的目的地，目的地类似于邮筒。同时解放发送者，使发送者能够继续进行其他业务。

两种通用的 目的地：队列和主题。
队列：点对点模型

```
消息发送  -- 》 队列  --》接收者
```

主题：发布-订阅模型

```
							订阅者A
消息发送 -- > 主题   -->      订阅者B
						    订阅者C
```


# 使用jms发送消息基于activeMQ
Java消息服务（Java message service，JMS）是一个Java标准，定义了使用消息代理的通用API。
spring基于模板的抽象为JMS功能提供了支持，这个模板就是 jmstemplate。使用jmstemplate，能够非常容易地在消息生产方发送队列和主题，在接收方也非常容易接收消息。
但是发送消息还需要一个消息代理，这里采用activeMQ。

## 在spring中搭建消息代理
下载地址  免分
http://download.csdn.net/download/xiaofeienen/9754186

解压到D盘
![这里写图片描述](http://img.blog.csdn.net/20170518161321210?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


启动消息代理 打开这个bat文件
D:\apache-activemq-5.14.3\bin\win64\activemq.bat   


## 创建连接工厂

ConnectionFactory是用于产生到JMS服务器的链接的，Spring为我们提供了多个ConnectionFactory，有SingleConnectionFactory和CachingConnectionFactory。SingleConnectionFactory对于建立JMS服务器链接的请求会一直返回同一个链接，并且会忽略Connection的close方法调用。CachingConnectionFactory继承了SingleConnectionFactory，所以它拥有SingleConnectionFactory的所有功能，同时它还新增了缓存功能，它可以缓存Session、MessageProducer和MessageConsumer。
```
<bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory"/>  
```
Spring提供的ConnectionFactory只是Spring用于管理ConnectionFactory的，真正产生到JMS服务器链接的ConnectionFactory还得是由JMS服务厂商提供，并且需要把它注入到Spring提供的ConnectionFactory中。我们这里使用的是ActiveMQ实现的JMS，所以在我们这里真正的可以产生Connection的就应该是由ActiveMQ提供的ConnectionFactory。所以定义一个ConnectionFactory的完整代码应该如下所示

```
<bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">  
    <property name="brokerURL" value="tcp://192.168.7.234:61616"/>  
</bean>  
  
<!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->  
<bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">  
    <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->  
    <property name="targetConnectionFactory" ref="targetConnectionFactory"/>  
</bean>  
```

ActiveMQ为我们提供了一个PooledConnectionFactory，通过往里面注入一个ActiveMQConnectionFactory可以用来将Connection、Session和MessageProducer池化，这样可以大大的减少我们的资源消耗。当使用PooledConnectionFactory时，我们在定义一个ConnectionFactory时应该是如下配置

```
<bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">
		<property name="targetConnectionFactory" ref="pooledConnectionFactory"/>
</bean> 
	
<bean id="pooledConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory">  
	    <property name="connectionFactory" ref="targetConnectionFactory"/>  
	    <property name="maxConnections" value="10"/>  
</bean> 
	
<bean id="targetConnectionFactory" class="org.apache.activemq.spring.ActiveMQConnectionFactory">
		<property name="brokerURL" value="tcp://192.168.7.234:61616"/>
</bean>
```

## 配置生产者
生产者负责产生消息并发送到JMS服务器
利用Spring为我们提供的JmsTemplate类来处理消息发送 对于消息发送者而言，它在发送消息的时候要知道自己该往哪里发，为此，我们在定义JmsTemplate的时候需要往里面注入一个Spring提供的ConnectionFactory对象

```
<!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->  
<bean id="senderJmsTemplate" class="org.springframework.jms.core.JmsTemplate">  
    <!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->  
    <property name="connectionFactory" ref="connectionFactory"/>  
    <!-- true/订阅模式发送，false/点对点的模式发送 -->
	<property name="pubSubDomain" value="false"/>
</bean> 
```

## 配置目的地
用JmsTemplate进行消息发送的时候，我们需要知道消息发送的目的地(destination)

```
 <!-- 设置两个目的地，一个是队列形式，一个是主题模式（订阅/发布） -->
    <bean id="myQueue" class="org.apache.activemq.command.ActiveMQQueue">
    	<!-- 指定队列名称 -->
    	<constructor-arg>
    		<value>myQueue</value>
    	</constructor-arg>
    </bean>
    
    <bean id="myTopic" class="org.apache.activemq.command.ActiveMQTopic">
    	<!-- 指定主题名称 -->
    	<constructor-arg>
    		<value>myTopic</value>
    	</constructor-arg>
    </bean>
```

## 定义发送消息的接口与实现类
接口类
```
package com.jx.spring.jms.server;

public interface AlertService {
	void sendUserMsg(User user);
}
```
实现类

```
package com.jx.spring.jms.server;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;

import org.springframework.jms.core.JmsOperations;
import org.springframework.jms.core.MessageCreator;

public class AlertServiceImpl implements AlertService{
	
	private JmsOperations jmsOperations;			//注入jms模板
	
	@Override
	public void sendUserMsg(final String dest , final String msg) {
		jmsOperations.send(				//发送消息
				dest,			//指定目的地
				new MessageCreator() {
					@Override
					public Message createMessage(Session session) throws JMSException {
						return session.createTextMessage(msg);		//创建消息
					}
				}
			);
	}

	public JmsOperations getJmsOperations() {
		return jmsOperations;
	}

	public void setJmsOperations(JmsOperations jmsOperations) {
		this.jmsOperations = jmsOperations;
	}
}
```
### 为发送消息接口注入 生产者

```
	<!-- 注入AlertServiceImpl -->
    <bean id="alertService" class="com.jx.spring.jms.server.AlertServiceImpl">
    	<property name="jmsOperations" ref="sendTmsTemplate"/>
    </bean>
```
### 编写测试发送消息入口类

```
	public static void main(String[] args) {
		
		ClassPathXmlApplicationContext cxt = new ClassPathXmlApplicationContext("server.xml");
		AlertServiceImpl alert = (AlertServiceImpl) cxt.getBean("alertService");
		String dest = "myQueue";
		String msg = "hello";
		alert.sendUserMsg(dest , msg);
		
	}
```

## 配置消费者
生产者往指定目的地Destination发送消息后，接下来就是消费者对指定目的地的消息进行消费了。
消费者是如何知道有生产者发送消息到指定目的地Destination了呢？这是通过Spring为我们封装的消息监听容器MessageListenerContainer实现的，它负责接收信息，并把接收到的信息分发给真正的MessageListener进行处理。每个消费者对应每个目的地都需要有对应的MessageListenerContainer。对于消息监听容器而言，除了要知道监听哪个目的地之外，还需要知道到哪里去监听，也就是说它还需要知道去监听哪个JMS服务器，这是通过在配置MessageConnectionFactory的时候往里面注入一个ConnectionFactory来实现的。所以我们在配置一个MessageListenerContainer的时候有三个属性必须指定，一个是表示从哪里监听的ConnectionFactory；一个是表示监听什么的Destination；一个是接收到消息以后进行消息处理的MessageListener。
Spring一共为我们提供了两种类型的MessageListenerContainer，SimpleMessageListenerContainer和DefaultMessageListenerContainer。
SimpleMessageListenerContainer会在一开始的时候就创建一个会话session和消费者Consumer，并且会使用标准的JMS MessageConsumer.setMessageListener()方法注册监听器让JMS提供者调用监听器的回调函数。它不会动态的适应运行时需要和参与外部的事务管理。兼容性方面，它非常接近于独立的JMS规范，但一般不兼容Java EE的JMS限制。
大多数情况下我们还是使用的DefaultMessageListenerContainer，跟SimpleMessageListenerContainer相比，DefaultMessageListenerContainer会动态的适应运行时需要，并且能够参与外部的事务管理。它很好的平衡了对JMS提供者要求低、先进功能如事务参与和兼容Java EE环境。
### 定义处理消息的MessageListener
```
package com.jx.spring.jms.client;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class MyMessageListener implements MessageListener{

	@Override
	public void onMessage(Message msg) {
		TextMessage text = (TextMessage) msg;
		System.out.println("client 1 收到一条消息");
		try {
			System.out.println(text.getText());
		} catch (JMSException e) {
			e.printStackTrace();
		}
	}
	
}
```

要定义处理消息的MessageListener我们只需要实现JMS规范中的MessageListener接口就可以了。MessageListener接口中只有一个方法onMessage方法，当接收到消息的时候会自动调用该方法

### 配置消息监听

```
	<bean id="myMessageListener" class="com.jx.spring.jms.client.MyMessageListener"/>
```
### 配置消息监听容器
这里省略connectionFactory 、destination        这两个需要与发送端保持一致即可
```
	<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">  
	    <property name="connectionFactory" ref="connectionFactory" />  
	    <property name="destination" ref="myQueue" />
	    <property name="messageListener" ref="myMessageListener" />
	</bean>
```
### 编写接收端

```
public static void main(String[] args) {
		
		ClassPathXmlApplicationContext cxt = new ClassPathXmlApplicationContext("client.xml");
		System.out.println("接收者1启动");
	}
```

## 测试结果
运行发送端测试类，与接收端测试类 (先后运行哪个端都可以)
控制台输出

![这里写图片描述](http://img.blog.csdn.net/20170522112746958?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170522112806857?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


# 附上完整的配置与Java类
## 发送端spring配置文件

```
<!-- 配置连接工厂，指定消息代理的URL -->
	
	<bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">
		<property name="targetConnectionFactory" ref="pooledConnectionFactory"/>
	</bean> 
	
	<bean id="pooledConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory">  
	    <property name="connectionFactory" ref="targetConnectionFactory"/>  
	    <property name="maxConnections" value="10"/>  
	</bean> 
	
	<bean id="targetConnectionFactory" class="org.apache.activemq.spring.ActiveMQConnectionFactory">
		<property name="brokerURL" value="tcp://192.168.7.234:61616"/>
	</bean>
    
    
    <!-- 配置发送者 -->
    <bean id="sendTmsTemplate" class="org.springframework.jms.core.JmsTemplate">  
	    <!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->  
	    <property name="connectionFactory" ref="connectionFactory"/>
	    <!-- true/订阅模式发送，false/点对点的模式发送 -->
	    <property name="pubSubDomain" value="false"/>
	</bean>
    
    <!-- 设置两个目的地，一个是队列形式，一个是主题模式（订阅/发布） -->
    <bean id="myQueue" class="org.apache.activemq.command.ActiveMQQueue">
    	<!-- 指定队列名称-->
    	<constructor-arg>
    		<value>myQueue</value>
    	</constructor-arg>
    </bean>
    
    <bean id="myTopic" class="org.apache.activemq.command.ActiveMQTopic">
    	<!-- 指定主题名称 -->
    	<constructor-arg>
    		<value>myTopic</value>
    	</constructor-arg>
    </bean>
    
    <!-- 注入AlertServiceImpl -->
    <bean id="alertService" class="com.jx.spring.jms.server.AlertServiceImpl">
    	<property name="jmsOperations" ref="sendTmsTemplate"/>
    </bean>
```

## 发送端接口与实现类
接口
```
public interface AlertService {
	void sendUserMsg(String dest , String msg);
}
```
实现类

```
package com.jx.spring.jms.server;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;
import org.springframework.jms.core.JmsOperations;
import org.springframework.jms.core.MessageCreator;

public class AlertServiceImpl implements AlertService{
	private JmsOperations jmsOperations;			//注入jms模板
	@Override
	public void sendUserMsg(final String dest , final String msg) {
		jmsOperations.send(				//发送消息
				dest,			//指定目的地
				new MessageCreator() {
					@Override
					public Message createMessage(Session session) throws JMSException {
						return session.createTextMessage(msg);		//创建消息
					}
				}
			);
	}
	public JmsOperations getJmsOperations() {
		return jmsOperations;
	}
	public void setJmsOperations(JmsOperations jmsOperations) {
		this.jmsOperations = jmsOperations;
	}
}
```

## 发送端发送测试类

```
public static void main(String[] args) {
		
		ClassPathXmlApplicationContext cxt = new ClassPathXmlApplicationContext("server.xml");
		AlertServiceImpl alert = (AlertServiceImpl) cxt.getBean("alertService");
		String dest = "myQueue";
		String msg = "hello";
		alert.sendUserMsg(dest , msg);
		
	}
```

## 接收端配置文件

```
<!-- 配置连接工厂，指定消息代理的URL -->
	
	<bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">
		<property name="targetConnectionFactory" ref="pooledConnectionFactory"/>
	</bean> 
	
	<bean id="pooledConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory">  
	    <property name="connectionFactory" ref="targetConnectionFactory"/>  
	    <property name="maxConnections" value="10"/>  
	</bean>
	
	<bean id="targetConnectionFactory" class="org.apache.activemq.spring.ActiveMQConnectionFactory">
		<property name="brokerURL" value="tcp://192.168.7.234:61616"/>
	</bean>
    
    <!-- 设置两个目的地，一个是队列形式，一个是主题模式（订阅/发布） -->
    <bean id="myQueue" class="org.apache.activemq.command.ActiveMQQueue">
    	<!-- 指定队列名称 -->
    	<constructor-arg>
    		<value>myQueue</value>
    	</constructor-arg>
    </bean>
    
    <bean id="myTopic" class="org.apache.activemq.command.ActiveMQTopic">
    	<!-- 指定主题名称 -->
    	<constructor-arg>
    		<value>myTopic</value>
    	</constructor-arg>
    </bean>
    
	<!-- 消息监听器 -->  
	<bean id="myMessageListener" class="com.jx.spring.jms.client.MyMessageListener"/>      
	 
	<!-- 消息监听容器 -->  
	<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">  
	    <property name="connectionFactory" ref="connectionFactory" />  
	    <property name="destination" ref="myQueue" />
	    <property name="messageListener" ref="myMessageListener" />
	</bean>
```
## 接收端监听类

```
package com.jx.spring.jms.client;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class MyMessageListener implements MessageListener{

	@Override
	public void onMessage(Message msg) {
		TextMessage text = (TextMessage) msg;
		System.out.println("client 1 收到一条消息");
		try {
			System.out.println(text.getText());
		} catch (JMSException e) {
			e.printStackTrace();
		}
	}
	
}


```
## 接收端测试类

```
public static void main(String[] args) {
		
		ClassPathXmlApplicationContext cxt = new ClassPathXmlApplicationContext("client.xml");
		System.out.println("接收者1启动");
	}
```


## 其他说明
1  发送端，接收端 连接工厂保持一致，目的地设置保持一致
2  若要发送订阅/发布模式的，修改发送者  pubSubDomain 改成 true

```
	 <!-- 配置发送者 -->
    <bean id="sendTmsTemplate" class="org.springframework.jms.core.JmsTemplate">  
	    <!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->  
	    <property name="connectionFactory" ref="connectionFactory"/>
	    <!-- true/订阅模式发送，false/点对点的模式发送 -->
	    <property name="pubSubDomain" value="false"/>
	</bean>
```

3  MessageListener  是JMS标准的，纯粹的接收消息使用
   SessionAwareMessageListener 是Spring为我们提供的，它不是标准的JMS MessageListener，可以在接收消息的时候回复一条消息给发送端