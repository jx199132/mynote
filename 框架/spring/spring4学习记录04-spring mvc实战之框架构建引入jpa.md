# 本章目标
引入jpa
新建一个User类,数据库自动生成一个user表

## 引入相关jar
打开pom文件 加入 依赖jar
hibernate版本是 4.2.0，加入在pom properties节点中

```
<hibernate-version>4.2.0.Final</hibernate-version>
```

```
		<!-- spring jpa所需要的jar -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${spring-version}</version>
		</dependency>
		<!-- 引入  jpa具体实现 的jar -->
		<dependency>
		    <groupId>org.hibernate</groupId>
		    <artifactId>hibernate-core</artifactId>
		    <version>${hibernate-version}</version>
		</dependency>
		<dependency>
		    <groupId>org.hibernate</groupId>
		    <artifactId>hibernate-entitymanager</artifactId>
		    <version>${hibernate-version}</version>
		</dependency>
		<dependency>
		    <groupId>org.hibernate.javax.persistence</groupId>
		    <artifactId>hibernate-jpa-2.0-api</artifactId>
		    <version>1.0.1.Final</version>
		</dependency>
		<!-- 引入 jpa 所需要的数据源连接池 -->
		<dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
			<version>1.4</version>
		</dependency>
		<!-- 数据库mysql -->
	    <dependency>
	         <groupId>mysql</groupId>
	         <artifactId>mysql-connector-java</artifactId>
	         <version>5.1.25</version>
	     </dependency>
```

## 新建一个配置文件 custom.properties
文件地址在src/main/resources下
![这里写图片描述](http://img.blog.csdn.net/20170426155448179?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

内容如下
配置了jdbc需要的信息，jpa需要的信息
```
#jdbc
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/springinaction?useUnicode=true&characterEncoding=UTF-8
jdbc.username=root
jdbc.password=root

#jpa
jpa.showsql=true
jpa.database=MYSQL
jpa.generateDdl=true
```
## 配置spring的配置文件 applicationContext.xml

在上一章中spring的配置文件一个bean都没有配置是空的.
这里先说明spring 如何配置

### 引入 上面自定义的配置文件 custom.properties

```
<context:property-placeholder location="classpath:custom.properties"/>
```
### 配置entityManagerFactory
jpa应用需要使用entityManagerFactory实现类来获取entityManager实例.
jpa中定义了两种类型的实体管理器,分别为**应用程序管理类型**和**容器管理类型**
这两种实体管理器工厂由spring工厂Bean创建,这里采用容器管理类型. 应用程序管理类型这里就不多说了.
```
<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<!-- 配置jpa 使用的数据源，这里采用dbcp连接池 -->
		<property name="dataSource" ref="dataSource"/>
		<!-- 配置引入哪一个厂商的jpa实现适配器，这里使用hibernate -->
		<property name="jpaVendorAdapter" ref="hibernateAdapter"/>
		<!-- 配置扫描bean的包 -->
		<property name="packagesToScan" value="com.jx.springinaction.bean"/>
	</bean>
```
配置完毕entityManagerFactory之后可以看到需要数据源，适配器，配置扫描的实体Bean的package.
### 配置数据源
这里的变量值对应上面custom的文件中的值
```
	<!-- 数据源 -->
  	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	    <property name="driverClassName" value="${jdbc.driverClassName}"/>
	    <property name="url" value="${jdbc.url}"/>
	    <property name="username" value="${jdbc.username}"/>
	    <property name="password" value="${jdbc.password}"/>
	    <property name="maxActive" value="20"/>
		<property name="validationQuery" value="SELECT 'x'"/>
        <property name="testWhileIdle" value="true"/>
        <property name="testOnBorrow" value="false"/>
        <property name="testOnReturn" value="true"/>
		<property name="timeBetweenEvictionRunsMillis" value="600000"/>
		<property name="minEvictableIdleTimeMillis" value="3600000"/>
	</bean>
```
### 配置jpa实现适配器

```
<!-- jpa实现适配器 -->
	<bean id="hibernateAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
		<!-- 没什么好说的 是否开启 控制台打印sql语句 -->
		<property name="showSql" value="${jpa.showsql}"/>
		<!-- 方言  ： DB2, DERBY, H2, HSQL, INFORMIX, MYSQL, ORACLE, POSTGRESQL, SQL_SERVER, SYBASE -->
		<property name="database" value="${jpa.database}"/>
		<!-- true or false  相当于 hibernate中的  hbm2ddl  是否开启 update -->
		<property name="generateDdl" value="${jpa.generateDdl}"/>
	</bean>
```
### 配置Jpa事务管理器 

```
	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>
```
### 开启事物管理注解

```
<tx:annotation-driven transaction-manager="transactionManager"/>
```

### 完整的application.xml内容

```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
          http://www.springframework.org/schema/context    
      	  http://www.springframework.org/schema/context/spring-context-4.2.xsd
      	  http://www.springframework.org/schema/data/jpa 
      	  http://www.springframework.org/schema/data/jpa/spring-jpa-1.0.xsd
      	   http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
          ">
	
    <!-- 引入属性文件 -->
    <context:property-placeholder location="classpath:custom.properties"/>
	
	<!-- jpa start-->
	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<!-- 配置jpa 使用的数据源，这里采用dbcp连接池 -->
		<property name="dataSource" ref="dataSource"/>
		<!-- 配置引入哪一个厂商的jpa实现适配器，这里使用hibernate -->
		<property name="jpaVendorAdapter" ref="hibernateJpaVendorAdapter"/>
		<!-- 配置扫描bean的包 -->
		<property name="packagesToScan" value="com.jx.springinaction.bean"/>
		<!-- <property name="persistenceUnitName" value="springinaction"></property> -->
	</bean>
	
	<!-- 数据源 -->
  	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	    <property name="driverClassName" value="${jdbc.driverClassName}"/>
	    <property name="url" value="${jdbc.url}"/>
	    <property name="username" value="${jdbc.username}"/>
	    <property name="password" value="${jdbc.password}"/>
	    <property name="maxActive" value="20"/>
		<property name="validationQuery" value="SELECT 'x'"/>
        <property name="testWhileIdle" value="true"/>
        <property name="testOnBorrow" value="false"/>
        <property name="testOnReturn" value="true"/>
		<property name="timeBetweenEvictionRunsMillis" value="600000"/>
		<property name="minEvictableIdleTimeMillis" value="3600000"/>
	</bean>
	
	<!-- jpa实现适配器 -->
	<bean id="hibernateJpaVendorAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
		<!-- 没什么好说的 是否开启 控制台打印sql语句 -->
		<property name="showSql" value="${jpa.showsql}"/>
		<!-- 方言  ： DB2, DERBY, H2, HSQL, INFORMIX, MYSQL, ORACLE, POSTGRESQL, SQL_SERVER, SYBASE -->
		<property name="database" value="${jpa.database}"/>
		<!-- true or false  相当于 hibernate中的  hbm2ddl  是否开启 update -->
		<property name="generateDdl" value="${jpa.generateDdl}"/>
	</bean>
	
	<!-- 配置Jpa事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>
    
    <!-- 开启事物管理注解  -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

### 回顾一下所引入的jar
spring-orm ： 可以看到jpa全部来至于此包   LocalContainerEntityManagerFactoryBean就是如此

使用的是hibernate的jpa实现，那么需要引入hibernate的jar包
http://hibernate.org/orm/downloads/  
明确说明jpa需要引入的jar包  core 和 entitymanager
hibernate-core   
hibernate-entitymanager
![这里写图片描述](http://img.blog.csdn.net/20170426161228702?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
hibernate-jpa-2.0-api : 这里有版本对应关系，如果用4.3以下采用2.0 而不要采用2.1
![这里写图片描述](http://img.blog.csdn.net/20170426161335252?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

commons-dbcp  :  连接池 也可以选择其他的不一定非要这个.
mysql-connector-java  :  数据库驱动没什么好说的.

## 新建User实体Bean

```
package com.jx.springinaction.bean;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table
public class User {
	@Id
	private String id;
	@Column
	private String name;
	
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
}
```

## 新建一个数据库 名称 跟配置文件中一样

## 测试
### 引入spring-test
在pom文件中

```
	<!-- spring test -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring-version}</version>
		</dependency>
```

### 新建测试类

```
package inaction;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;


@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={"classpath:applicationContext.xml"})
public class Init {
	
	@Test
	public void test(){
		
	}
}
```
直接运行，可以看到数据库生成user表
![这里写图片描述](http://img.blog.csdn.net/20170426162239575?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

