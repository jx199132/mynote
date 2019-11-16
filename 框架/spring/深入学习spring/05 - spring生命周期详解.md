# 开篇

第三篇 简单过了一些 spring 的源码，了解spring容器的启动，加载bean ， 实例化，这里 以应用的角度 详解 bean的生命周期



# 生命周期流程图

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191029105445.png)





# 代码

## 结构

### 结构图

```java
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── jx
│   │   │           └── spring
│   │   │               └── bean
│   │   │                   └── cycle
│   │   │                       ├── BeanLifeCycle.java
│   │   │                       ├── MyBeanFactoryPostProcessor.java
│   │   │                       ├── MyBeanPostProcessor.java
│   │   │                       ├── MyInstantiationAwareBeanPostProcessor.java
│   │   │                       └── Person.java
│   │   └── resources
│   │       └── application.xml

```

## 文件

### pom.xml

```xml
<dependencies>
        <!-- 会自动引入core、bean等模块 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.4</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>
```

### Application.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">

    <bean id="beanPostProcessor" class="com.jx.spring.bean.cycle.MyBeanPostProcessor"/>

    <bean id="instantiationAwareBeanPostProcessor" class="com.jx.spring.bean.cycle.MyInstantiationAwareBeanPostProcessor"/>

    <bean id="beanFactoryPostProcessor" class="com.jx.spring.bean.cycle.MyBeanFactoryPostProcessor"/>

    <bean id="person" class="com.jx.spring.bean.cycle.Person" init-method="myInit"
          destroy-method="myDestory" scope="singleton">
        <property name="address" value="武汉市"/>
        <property name="phone" value="123456"/>
        <property name="name" value="九霄"/>
    </bean>
</beans>
```



### Person

```java

import lombok.ToString;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;

@ToString
public class Person implements BeanFactoryAware, BeanNameAware, InitializingBean, DisposableBean {
    private String name;
    private String address;
    private int phone;
    private BeanFactory beanFactory;
    private String beanName;

    public Person() {
        System.out.println("【构造器】调用Person的构造器实例化");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("【注入属性】name");
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        System.out.println("【注入属性】address");
        this.address = address;
    }

    public int getPhone() {
        return phone;
    }

    public void setPhone(int phone) {
        System.out.println("【注入属性】phone");
        this.phone = phone;
    }

    // 这是BeanFactoryAware接口方法
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("【BeanFactoryAware接口】调用setBeanFactory方法");
        this.beanFactory = beanFactory;
    }

    // 这是BeanNameAware接口方法
    @Override
    public void setBeanName(String s) {
        System.out.println("【BeanNameAware接口】调用setBeanName方法");
        this.beanName = s;
    }

    // 这是DiposibleBean接口方法
    @Override
    public void destroy() throws Exception {
        System.out.println("【DiposibleBean接口】调用destroy方法");
    }

    // 这是InitializingBean接口方法
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("【InitializingBean接口】调用afterPropertiesSet方法");
    }

    // 通过<bean>的init-method属性指定的初始化方法
    public void myInit() {
        System.out.println("【init-method】调用<bean>的init-method属性指定的初始化方法");
    }

    // 通过<bean>的destroy-method属性指定的初始化方法
    public void myDestory() {
        System.out.println("【destroy-method】调用<bean>的destroy-method属性指定的初始化方法");
    }

}
```

### MyBeanFactoryPostProcessor

```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor{
    public MyBeanFactoryPostProcessor() {
        super();
        System.out.println("这是BeanFactoryPostProcessor实现类构造器！！");
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println("BeanFactoryPostProcessor调用postProcessBeanFactory方法");
        BeanDefinition bd = configurableListableBeanFactory.getBeanDefinition("person");
        bd.getPropertyValues().addPropertyValue("phone", "110");
    }
}
```

### MyBeanPostProcessor

```java
public class MyBeanPostProcessor implements BeanPostProcessor{
    public MyBeanPostProcessor(){
        System.out.println("这是BeanPostProcessor实现类构造器！！");
    }
    @Override
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        System.out.println("BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改");
        return o;
    }

    @Override
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        System.out.println("BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改");
        return o;
    }
}
```

### MyInstantiationAwareBeanPostProcessor

```java
public class MyInstantiationAwareBeanPostProcessor implements 	InstantiationAwareBeanPostProcessor {

    public MyInstantiationAwareBeanPostProcessor() {
        super();
        System.out
                .println("这是InstantiationAwareBeanPostProcessorAdapter实现类构造器！！");
    }

    // 接口方法、实例化Bean之前调用
    @Override
    public Object postProcessBeforeInstantiation(Class beanClass,
                                                 String beanName) throws BeansException {
        System.out
                .println("InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法");
        return null;
    }

    // 接口方法、实例化Bean之后调用
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
            throws BeansException {
        System.out
                .println("InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法");
        return bean;
    }

    // 接口方法、设置某个属性时调用
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs,
                                                    PropertyDescriptor[] pds, Object bean, String beanName)
            throws BeansException {
        System.out
                .println("InstantiationAwareBeanPostProcessor调用postProcessPropertyValues方法");
        return pvs;
    }
}
```

### 测试类 - BeanLifeCycle

```java
public class BeanLifeCycle {

    public static void main(String[] args) {

        System.out.println("现在开始初始化容器");

        ClassPathXmlApplicationContext factory = new ClassPathXmlApplicationContext("application.xml");
        System.out.println("容器初始化成功");
        System.out.println("获取Bean");
        Person person = factory.getBean("person",Person.class);
        System.out.println(person);

        System.out.println("现在开始关闭容器！");
        factory.registerShutdownHook();
    }
}
```





## 测试结果

```
现在开始初始化容器
      这是BeanFactoryPostProcessor实现类构造器！！
      BeanFactoryPostProcessor调用postProcessBeanFactory方法
      这是BeanPostProcessor实现类构造器！！
      这是InstantiationAwareBeanPostProcessorAdapter实现类构造器！！
      InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法
      【构造器】调用Person的构造器实例化
      InstantiationAwareBeanPostProcessor调用postProcessPropertyValues方法
      【注入属性】address
      【注入属性】phone
      【注入属性】name
      【BeanNameAware接口】调用setBeanName方法
      【BeanFactoryAware接口】调用setBeanFactory方法
      BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改
      【InitializingBean接口】调用afterPropertiesSet方法
      【init-method】调用<bean>的init-method属性指定的初始化方法
      BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改
      InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法
容器初始化成功


获取Bean
    Person(name=九霄, address=武汉市, phone=110)


现在开始关闭容器！
    【DiposibleBean接口】调用destroy方法
    【destroy-method】调用<bean>的destroy-method属性指定的初始化方法
```



# 生命周期流程详解



bean的生命周期实际上只有四个：

- 实例化 Instantiation
- 属性赋值 Populate
- 初始化 Initialization
- 销毁 Destruction



spring 在bean 的 四个生命周期中 插入了一些扩展点。扩展点又分为 **影响单个bean** 和 **影响多个bean **两种



从输出中已经非常清楚了的体现了 整个 spring 容器的启动 ，以及 bean 对象的 生命周期，按照从前到后的顺序进行说明



## BeanFactoryPostProcessor接口

严格来说这个不属于 spring bean生命周期之中



该接口只有一个方法	postProcessBeanFactory	在BeanFactory标准初始化之后调用，这时所有的bean定义已经保存加载到beanFactory，但是bean的实例还未创建



可以用来改变Bean的定义，例如在配置文件中定义的 Person 注入的  电话是 123456 ，这里修改了Bean的定义，最后生成的 person 的 电话是  110



bean的定义

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191029141127.png)



工厂中修改 bean的定义

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191029141153.png)



输出结果：Person(name=九霄, address=武汉市, phone=110)



## 影响多个bean的接口

影响多个bean的接口 有两个 **InstantiationAwareBeanPostProcessor** 和 **BeanPostProcessor**



假设有 Student 和 Person 两个 bean，这两个bean 不需要继承 上面所述两个接口，这两个接口都会在 bean 生命周期中相应的 扩展点进行调用

```
public class Person{
    private String name;
    private String address;
    private int phone;
    private BeanFactory beanFactory;
    private String beanName;
}    

public class Student{
    private String name;
    private String address;
    private int phone;
    private BeanFactory beanFactory;
    private String beanName;
}

<bean id="person" class="com.jx.spring.bean.cycle.Person" scope="singleton">
        <property name="address" value="武汉市"/>
        <property name="phone" value="123456"/>
        <property name="name" value="九霄"/>
</bean>

<bean id="student" class="com.jx.spring.bean.cycle.Student" scope="singleton">
        <property name="id" value="1"/>
        <property name="code" value="123456"/>
</bean>
```



输出如下

```
InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法
构造 Person
InstantiationAwareBeanPostProcessor调用postProcessPropertyValues方法
InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法

InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法
构造 Student
InstantiationAwareBeanPostProcessor调用postProcessPropertyValues方法
InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法
```



InstantiationAwareBeanPostProcessor作用于实例化阶段的前后，BeanPostProcessor作用于初始化阶段的前后。正好和第一、第三个生命周期阶段对应。通过图能更好理解：

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191029151249.png)



### InstantiationAwareBeanPostProcessor

```java
// 接口方法、实例化Bean之前调用
    @Override
    public Object postProcessBeforeInstantiation(Class beanClass, String beanName) throws BeansException {
        System.out .println("InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法");
        return null;
    }

    // 接口方法、实例化Bean之后调用
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法");
        return bean;
    }

    // 接口方法、设置某个属性时调用
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {
        System.out.println("InstantiationAwareBeanPostProcessor调用postProcessPropertyValues方法");
        return pvs;
    }
```



### BeanPostProcessor

```java
@Override
public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        System.out.println("BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改");
        return o;
}

@Override
public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        System.out.println("BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改");
        return o;
}
```



## 只影响一个bean的接口

只影响一个bean的接口 与 上面影响多个bean的接口不同， 需要 bean 实现相应接口，才会在 该 bean的生命周期扩展点中起作用



### BeanFactoryAware

通过这个接口 可以 拿到 beanFactory 信息

```java
// 这是BeanFactoryAware接口方法
@Override
public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
	System.out.println("【BeanFactoryAware接口】调用setBeanFactory方法");
	this.beanFactory = beanFactory;
}
```



### BeanNameAware

通过这个接口可以拿到 当前 bean的名称

```java
// 这是BeanNameAware接口方法
@Override
public void setBeanName(String s) {
	System.out.println("【BeanNameAware接口】调用setBeanName方法");
	this.beanName = s;
}
```



### InitializingBean

实现这个接口的方法 在 bean对象 注入属性之后调用，可以用来检查 一些重要字段是否 注入了值

```java
// 这是InitializingBean接口方法
@Override
	public void afterPropertiesSet() throws Exception {
	System.out.println("【InitializingBean接口】调用afterPropertiesSet方法");
}
```

还可以指定一个 init-method 方法

```java
// 通过xml 配置 <bean>的init-method属性指定的初始化方法
public void myInit() {
	System.out.println("【init-method】调用<bean>的init-method属性指定的初始化方法");
}
```

先调用 afterPropertiesSet()  在 调用 init-method



### DisposableBean

销毁的时候调用 destory方法

```java
// 这是DiposibleBean接口方法
@Override
public void destroy() throws Exception {
	System.out.println("【DiposibleBean接口】调用destroy方法");
}
```



同样可以指定一个 destory-method方法

```java
// 通过<bean>的destroy-method属性指定的初始化方法
public void myDestory() {
	System.out.println("【destroy-method】调用<bean>的destroy-method属性指定的初始化方法");
}
```

先调用 destory ， 在调用  destory-method