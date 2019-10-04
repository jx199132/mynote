# 目标
###  测试jpa：实现通过jpa写入一条用户数据到数据库
###  借助springdata 实现自动化jpa

# 测试jpa
在上一章中只完成了 启动项目 通过User 实体Bean 自动建立表结构.
接下来 完成 编写测试类 写入数据到User中

## 配置spring文件

在上一章中 所有的配置文件全部在spring的一个配置文件当中，如果配置内容越来越多 不利于扩展，这里对spring配置文件进行切割

### 重命名applicationContext.xml 
在上一章中 applicationContext.xml 里面写入了很多Bean的配置.把这个文件重命名为 spring-jpa-data.xml
spring-jpa-data.xml 中的内容为

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
### 新建 spring-repository.xml
扫描 repository
```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
          http://www.springframework.org/schema/context    
      	  http://www.springframework.org/schema/context/spring-context-4.2.xsd
      	  http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
          ">
	
   <!-- 注解扫描repository -->
   <context:component-scan base-package="com.jx.springinaction.repository">
    	<context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

### 新建 spring-service.xml
扫描 service
```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
          http://www.springframework.org/schema/context    
      	  http://www.springframework.org/schema/context/spring-context-4.2.xsd
      	  http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
          ">
   <!-- 注解扫描Service -->
   <context:component-scan base-package="com.jx.springinaction.services">
    	<context:include-filter type="annotation" expression="org.springframework.stereotype.Service"/>
    </context:component-scan>
</beans>
```

### 新建 applicationContext.xml
汇总所有的spring 配置文件
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
	
	<import resource="classpath:spring-jpa-data.xml"/>
	<import resource="classpath:spring-repository.xml"/>
	<import resource="classpath:spring-service.xml"/>
	
	
    
    
</beans>
```
### 新建 repository
接口
```
package com.jx.springinaction.repository;

import com.jx.springinaction.bean.User;

public interface UserRepository {
	
	void save(User user);
	
}

```
实现类

```
package com.jx.springinaction.repository.impl;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

import org.springframework.stereotype.Repository;
import com.jx.springinaction.bean.User;
import com.jx.springinaction.repository.UserRepository;

@Repository
public class UserRepositoryImpl implements UserRepository{
	
	@PersistenceContext
	private EntityManager entityManager;			//注入entityManagerFactory

	@Override
	public void save(User user) {
		entityManager.persist(user);
	}
}
```

### 新建service
接口

```
package com.jx.springinaction.services;

import com.jx.springinaction.bean.User;

public interface UserService {
	void save(User user);
}
```
实现类

```
package com.jx.springinaction.services.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.jx.springinaction.bean.User;
import com.jx.springinaction.repository.UserRepository;
import com.jx.springinaction.services.UserService;

@Service
public class UserServiceImpl implements UserService{
	
	@Autowired
	private UserRepository userRepository;
	
	@Override
	@Transactional
	public void save(User user) {
		userRepository.save(user);
	}
}

```
### 编写测试类

```
package inaction;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.jx.springinaction.bean.User;
import com.jx.springinaction.services.UserService;


@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={"classpath:applicationContext.xml"})
public class Init {
	
	@Resource
	private UserService userService;
	
	@Test
	public void test(){
		User user = new User();
		user.setId("00xxxdsxffx");
		user.setName("张4");
		userService.save(user);
	}
}
```

## 测试执行test ok
![这里写图片描述](http://img.blog.csdn.net/20170427141531950?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 附上结构图
![这里写图片描述](http://img.blog.csdn.net/20170427141626685?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


# 借助springdata自动化处理jpa
## 引入需要的jar
打开pom文件加入依赖

```
	<!-- 引入spring data jpa -->
	    <dependency>
	        <groupId>org.springframework.data</groupId>
	        <artifactId>spring-data-jpa</artifactId>
	        <version>1.11.3.RELEASE</version>
	    </dependency>
```

## 修改spring-data-jpa.xml
加入
spring-data 会自动扫描这个包下目的所有继承了**JpaRepository**  的所有类
```
<jpa:repositories base-package="com.jx.springinaction.repository"></jpa:repositories>
```

## 修改UserRepository

1 继承JpaRepository
2 删掉 UserRepository的 save方法

这个接口只是一个空的接口，目的是为了统一所有Repository的类型，其接口类型使用了泛型，泛型参数中T代表实体类型，ID则是实体中id的类型。代码如下:

```
package com.jx.springinaction.repository;

import org.springframework.data.jpa.repository.JpaRepository;

import com.jx.springinaction.bean.User;

public interface UserRepository extends JpaRepository<User, String>{
	
}
```
## 删除UserRepositoryImpl实现类

![这里写图片描述](http://img.blog.csdn.net/20170427144846592?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## junit测试
测试类代码
```
package inaction;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.jx.springinaction.bean.User;
import com.jx.springinaction.services.UserService;


@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={"classpath:applicationContext.xml"})
public class Init {
	
	@Resource
	private UserService userService;
	
	@Test
	public void test(){
		User user = new User();
		user.setId("springdatajpa");
		user.setName("张4");
		userService.save(user);
	}
}
```
### 数据库
![这里写图片描述](http://img.blog.csdn.net/20170427144953889?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaml1eGlhbzE5OTEzMg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

