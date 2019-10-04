# 概念
远程调用与本地调用的区别在于距离，程序内部直接调用叫做本地调用，一个程序访问另外一个程序提供的服务就叫做远程调用。
类似于人与人之间的交流，两个人在会议室讨论一个话题，这种面对面的形式就是本地交流。而如果我们拿起电话跟另外一个城市的人打电话来交流，这就是远程调用(Remote procedure call , RPC)


# spring提供的远程调用模型(这里主要说明RMI)
| RPC模型 | 适用场景 |
| ------------- | -----|
| 远程方法调用(RMI) | 使用JRMP协议(基于TCP/IP)，不允许穿透防火墙，使用JAVA系列化方式，使用于任何JAVA应用之间相互调用 |
|Hessian或Burlap|使用HTTP协议，允许穿透防火墙，使用自己的系列化方式，支持JAVA、C++、.Net等跨语言使用,与Hessian相同，只是Hessian使用二进制传输，而Burlap使用XML格式传输|
|HTTP invoker|使用HTTP协议，允许穿透防火墙，使用JAVA系列化方式，但仅限于Spring应用之间使用，即调用者与被调用者都必须是使用Spring框架的应用|
|Jax-RPC和JAX-WS|访问/发布平台独立的，基于soap的web服务|

# spring中远程调用模型的使用
## RMI
RMI最初在jdk1.1被引入到Java中，它为Java开发者提供了一种强大的方法来实现 Java 程序之间的交互.

创建一个RMI服务需要以下几个步骤
1  编写一个服务实现类，类的方法必须抛出Java.rmi.RemoteException异常
2  创建一个继承于java.rmi.Remote的服务接口
3  运行RMI编译器，创建客户端stub类服务端skeleton类
4  启动一个RMI注册表，以便持有这些服务
5  在RMI注册表中注册服务
可以看到非常的麻烦

使用spring对RMI的支持，可以非常容易地构建你的分布式应用。在服务端，可以通过Spring的org.springframework.remoting.rmi.RmiServiceExporter可以暴露你的服务；在客户端，通过org.springframework.remoting.rmi.RmiProxyFactoryBean可以使用服务端暴露的服务，非常方便。这种C/S模型的访问方式，可以屏蔽掉RMI本身的复杂性，如服务端Skeleton和客户端Stub等的处理细节，这些对于服务开发和服务使用的人员来说，都是透明的，无需过度关注，而集中精力开发你的商业逻辑。


### 创建服务项目

附上项目结构图

![这里写图片描述](http://img.blog.csdn.net/20170518114410100?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 定义一个服务接口

```
package com.jx.spring.rpc.rmi.demo;

public interface CalcService {
    public int add(int a,int b);
}
```

#### 编写实现类

```
package com.jx.spring.rpc.rmi.demo;


public class CalcServiceImpl implements CalcService{

	public int add(int a, int b) {
		System.out.println(a + " + " + b + " = " + ( a + b ) );
		return a + b;
	}

}
```
#### 编写spring配置文件

```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
          http://www.springframework.org/schema/context    
      	  http://www.springframework.org/schema/context/spring-context-4.2.xsd
          ">
	
	<bean id="serviceExporter" class="org.springframework.remoting.rmi.RmiServiceExporter">  
        <property name="serviceName" value="myCalcService" />  
        <property name="service" ref="calcService" />  
        <property name="serviceInterface" value="com.jx.spring.rpc.rmi.demo.CalcService" />  
        <property name="registryPort" value="8080" />
        <property name="servicePort" value="8088" />
    </bean>  
  
    <bean id="calcService" class="com.jx.spring.rpc.rmi.demo.CalcServiceImpl" />
    
</beans>
```
#### 编写Main启动服务

```
import org.springframework.context.support.ClassPathXmlApplicationContext;


public class Server {
	
	
	public static void main(String[] args) {
		
		new ClassPathXmlApplicationContext("applicationContext.xml"); 
		
	}
	
}
```

### 创建客户端项目

附上项目结构图

![这里写图片描述](http://img.blog.csdn.net/20170518114703491?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 创建一个客户端接口，之类名称随意命名不需要跟客户端保持一致，但是  方法名必须一致，否则spring代理 无法 识别

```
package com.jx.spring.rpc.rmi.client;

public interface RMICalcService {
    public int add(int a,int b);
}
```
#### 编写配置文件

```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
          http://www.springframework.org/schema/context    
      	  http://www.springframework.org/schema/context/spring-context-4.2.xsd
          ">
          
	<bean id="myCalcService" class="org.springframework.remoting.rmi.RmiProxyFactoryBean">  
        <property name="serviceUrl" value="rmi://192.168.7.234:8080/myCalcService" />  
        <property name="serviceInterface" value="com.jx.spring.rpc.rmi.client.Xservice" />  
    </bean>
	
</beans>
```

#### 客户端调用服务端

```
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.jx.spring.rpc.rmi.client.RMICalcService;



public class RmiClient {
	public static void main(String[] args) {
		ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");  
		RMICalcService accountService = (RMICalcService) ctx.getBean("myCalcService");  
		int result = accountService.add(1, 2);
		System.out.println(result);
	}
}
```


## Hessian
http://blog.csdn.net/junshuaizhang/article/details/28441907
