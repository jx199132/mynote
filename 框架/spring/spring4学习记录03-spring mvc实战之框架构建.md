# 目的
构建spring+springmvc+springDataJpa+shiro完成一个简单的Demo
# 本章目标
新建一个maven web项目，集成spring+springmvc
新建一个contorller 请求控制层 传递一个参数，并且在jsp显示

# 项目构建
##新建java项目(这里建立的是maven项目)
![这里写图片描述](http://img.blog.csdn.net/20170426094347039?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**一直next**
![这里写图片描述](http://img.blog.csdn.net/20170426094547243?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170426094705660?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 项目进行编辑
选中项目右键进行编辑，选中3.0版本,jdk1.7,或者更高这里用的myeclipse2013 选不了1.8
![这里写图片描述](http://img.blog.csdn.net/20170426101053832?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 修改pom文件引入插件

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.jx</groupId>
  <artifactId>springinaction</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>springinaction Maven Webapp</name>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <finalName>springinaction</finalName>
    <plugins>
    	<plugin>
	        <groupId>org.apache.maven.plugins</groupId>
	        <artifactId>maven-compiler-plugin</artifactId>
	        <version>2.3.2</version>
	        <configuration>
	            <source>1.7</source>
	            <target>1.7</target>
	        </configuration>
	    </plugin>
    </plugins>
  </build>
</project>
```
上面是修改好的pom文件内容：1 引入编译插件 2 junit改成了4.12版本

这个时候项目可能报错,那么选中项目 右键，maven -> update一下就好了.

## 测试项目
发布项目到tomcat，启动tomcat，浏览器访问 项目  能够看到 hello world即可.
![这里写图片描述](http://img.blog.csdn.net/20170426102309107?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170426102317006?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

到此 一个最基本的 maven web项目就完成了.

# 引入spring+springmvc
## 修改pom文件引入jar
### 加入properties节点
这里加入全局的来控制spring版本和项目构建编码

```
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<spring-version>4.3.6.RELEASE</spring-version>
	</properties>
```
### 加入spring依赖jar包
下面图示是spring下载下来的所有jar包
![这里写图片描述](http://img.blog.csdn.net/20170426104733243?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
http://repo.spring.io/release/org/springframework/spring/  这里是spring的下载地址
这里不引入这么多，先引入最基本的jar包

```
		<!-- spring 核心包 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-beans</artifactId>
			<version>${spring-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${spring-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-expression</artifactId>
			<version>${spring-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${spring-version}</version>
		</dependency>
		<!-- spring web需要的jar -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring-version}</version>
		</dependency>
		<!-- spring mvc -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring-version}</version>
		</dependency>
		<!-- 引入 Java web需要的jar -->
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.0.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax</groupId>
			<artifactId>javaee-api</artifactId>
			<version>7.0</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>taglibs</groupId>
			<artifactId>standard</artifactId>
			<version>1.1.2</version>
		</dependency>
```

## 配置web.xml

**web3.0之后开始支持不需要web.xml，直接在Java类中通过继承AbstractAnnotationConfigDispatcherServletInitializer 覆盖几个方法来进行配置web项目这里不做说明**

最开始的web.xml如图
![这里写图片描述](http://img.blog.csdn.net/20170426105416266?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

修改为

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
		 xmlns="http://java.sun.com/xml/ns/javaee" 
		 xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    	version="3.0">
    
   
  <display-name>springinaction</display-name>
  
  
  <!-- 配置spring -->
  <listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <context-param>
      <param-name>contextConfigLocation</param-name>
      <!-- 指定spring配置文件 -->
      <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
    
  <!-- 配置spring mvc -->
  <servlet>
      <servlet-name>springmvc</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <init-param>
          <param-name>contextConfigLocation</param-name>
          <!-- 指定spring mvc配置文件 -->
          <param-value>classpath:spring-mvc.xml</param-value>
      </init-param>
      <load-on-startup>1</load-on-startup>
   </servlet>
   <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
   </servlet-mapping>
   
   
   
   <!-- 设置编码filter -->
   <filter>
       <filter-name>CharEncoding</filter-name>
       <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
       <async-supported>true</async-supported>
       <init-param>
           <param-name>encoding</param-name>
           <param-value>UTF-8</param-value>
       </init-param>
       <init-param>
           <param-name>forceEncoding</param-name>
           <param-value>true</param-value>
       </init-param>
   </filter>
   <filter-mapping>
       <filter-name>CharEncoding</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
  
</web-app>
```

## 配置spring核心配置文件

### 新建结构
最开始的目录结构
![这里写图片描述](http://img.blog.csdn.net/20170426110916866?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
加上test 和 Java (如果不会再myeclipse中建立，那么去外面的文件夹建立然后刷新项目)
![这里写图片描述](http://img.blog.csdn.net/20170426111353576?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

src/main/java 放Java代码
src/main/resources 放配置文件
src/main/test 放测试类
src/main/webapp 放jsp ，js，css等

### 配置spring的配置文件applicationContext.xml

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
	
</beans>
```
### 配置spring-mvc配置文件spring-mvc.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.2.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd">
	
	<!-- 开启mvc注解 -->
    <mvc:annotation-driven/>
    
    <!-- 注解扫描controller -->
    <context:component-scan base-package="com.jx.springinaction.controllers">
    	<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <!-- 配置jsp视图 -->
    <bean id="viewResolver"
        class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>  
    </bean>
    
</beans>
```
## 新建HomeController控制器

```
package com.jx.springinaction.controllers;

import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/")
public class HomeController {
	
	@RequestMapping("home")
	public String home(String name , HttpServletRequest request){
		System.out.println("hello " + name);
		request.setAttribute("name", name);
		return "home";
	}
	
}
```
##新建home.jsp
在WEB-INF下新建文件夹views,然后在建立home.jsp
![这里写图片描述](http://img.blog.csdn.net/20170426115035015?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    
    <title>My JSP 'home.jsp' starting page</title>
    
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
	<meta http-equiv="expires" content="0">    
	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
	<meta http-equiv="description" content="This is my page">
	<!--
	<link rel="stylesheet" type="text/css" href="styles.css">
	-->

  </head>
  
  <body>
    Tmy name is ${name}
  </body>
</html>

```
## 测试
启动tomcat，浏览器输入请求
![这里写图片描述](http://img.blog.csdn.net/20170426115444933?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

