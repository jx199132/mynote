# 开篇

在jdk发展到1.5版本的时候，支持了注解，各大框架开始支持注解。



spring 注解的 发展 可以分为：

- 1.x注解驱动启蒙时代

- 2.x 注解驱动过度时代
- 3.x注解驱动黄金时代
- 4.x注解驱动完善时代
- 5.x注解驱动当下时代



# spring 1.x注解启蒙时代

在1.x版本 spring 注解 在框架层面支持的寥寥无几 ，但是 spring 的 核心 bean  仍然需要使用 xml 的形式进行配置



# Spring 2.x注解过度时代

在2.x版本时代，spring 注解 提供了 对 bean 提供了相关的 注解

## @Required

用于bean属性setter方法，并表示受影响的bean属性必须在XML配置文件在配置时进行填充。否则，容器会抛出一个BeanInitializationException异常

```java
public class Student {
    private Integer age;
    private String name;
    
    @Required
    public void setAge(Integer age){
        this.age = age;
    }
    
    public Integer getAge(){
        return this.age;
    }
    
    public void setName(String name){
        this.name = name;
    }
    
    public String getName(){
        return this.name;
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context.xsd">

   <context:annotation-config/>

   <bean id="student" class="Student">
        <property name="name" value="Jim"/>
   </bean>
</beans>
```

上面没有设置 age ，在实例化的时候就会抛出异常



## @repository、@Service、@Controller 和 @Component 

@repository 表示这是一个DAO层的 bean

@Service 表示是一个 业务层的 bean

@controller 表示是一个 控制层 的 bean

@Component 表示这是一个 bean ，可以放在任何地方



## @Autowired、@Resource、@Qualifier

@Autowired 默认先按byType，如果发现找到多个bean，则继续按照byName方式比对，如果还有多个，则报出异常，要指定byName 则需要加上@Qualifier

如果要允许null 值，可以设置它的required属性为false，如：@Autowired(required=false) 

```java
@Autowired

@Qualifier("pean")

public Fruit fruit;
```





@Resource 默认按 byName自动注入,如果找不到再按byType找bean,如果还是找不到则抛异常，无论按byName还是byType如果找到多个，则抛异常



## Java 注解 @PostConstruct和@PreDestroy

从Java EE5规范开始，Servlet增加了两个影响Servlet生命周期的注解（Annotation）：@PostConstruct和@PreDestroy。这两个注解被用来修饰一个非静态的void()方法.而且这个方法不能有抛出异常声明。



被@PostConstruct修饰的方法会在构造函数之后，init()方法之前运行

虽然PreDestory字面意思是在destory之前运行，但是被@PreDestory修饰的方法会在destory方法运行之后运行



在spring 2.5 中开始也支持这两个注解 ， 前者可以用来代替 <bean init-method=""/>  后者 可以用来 代替 <bean destory-method=""/>



## 其他 requestMapping、requestParam等略



# Spring3 .x注解黄金时代

## @Configuration

@Configuration等价 与XML中配置beans，相当于Ioc容器。它的某个方法上如果注册了@Bean，就会作为这个Spring容器中的Bean，与xml中配置的bean意思一样



@Configuration派生于 @Componet注解，所以加上了@Configuration也相当于加上了 @Componet



```java
@Getter
@Setter
public class Student {
    private Integer age;
}

@Getter
@Setter
public class Teacher {
    private String name;
}

@Configuration
public class MyConfig {

    @Bean
    public Student getStudent(){
        Student student = new Student();
        student.setAge(10);
        return student;
    }
}


public class Main {
    public static void main(String[] args) {

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.jx.learn.annotation");
        Student student = applicationContext.getBean(Student.class);
        System.out.println(student.getAge());


        Teacher teacher = applicationContext.getBean(Teacher.class);
        System.out.println(teacher.getName());
    }
}


运行 结果  Student 获取 正常， Teacher 获取失败
```



## @ImportResource、@Import

@ImportResource来导入一个传统的spring.xml配置文件

@Import注解是引入带有@Configuration的java类

```java
@Getter
@Setter
public class Student {
    private Integer age;
}

@Getter
@Setter
public class Student {
    private Integer age;
}

@Configuration
public class MyConfig {

    @Bean
    public Student getStudent(){
        Student student = new Student();
        student.setAge(10);
        return student;
    }

    @Bean
    public String setName(){
        return "jack";
    }
}

@Configuration
@Import(MyConfig.class)
public class MyConfig2 {

    @Bean
    public Teacher getTeacher(Student student, String name){
        System.out.println(student.getAge());
        Teacher teacher = new Teacher();
        teacher.setName(name);
        teacher.setStudent(student);
        return teacher;
    }
}


public class Main {
    public static void main(String[] args) {

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.jx.learn.annotation");
        Student student = applicationContext.getBean(Student.class);
        System.out.println(student.getAge());


        Teacher teacher = applicationContext.getBean(Teacher.class);
        System.out.println(teacher.getName());
    }
}
```





```
@ImportResource("classpath:cons-injec.xml") //导入xml配置项
@Configuration
public class SoundSystemConfig {
		
}
```



## @lazy

@lazy 表示懒加载没什么好说的



## @Primary

@primary 自动装配时当出现多个Bean候选者时，被注解为@Primary的Bean将作为首选者，否则将抛出异常

```java
@Component  
public class Apple implements Fruit{  
  
    @Override  
    public String hello() {  
        return "我是苹果";  
    }  
}
 
@Component  
@Primary
public class Pear implements Fruit{  
  
    @Override  
    public String hello(String lyrics) {  
        return "梨子";  
    }  
}
 
public class FruitService { 
  
  //Fruit有2个实例子类，因为梨子用@Primary，那么会使用Pear注入
    @Autowired  
    private Fruit fruit;  
  
    public String hello(){  
        return fruit.hello();  
    }  
}
```



## @PathVariable

```java
@Controller 
@RequestMapping("/owners/{a}") 
public class RelativePathUriTemplateController { 
  @RequestMapping("/pets/{b}") 
  public void findPet(@PathVariable("a") String a,@PathVariable String b, Model model) {     
    // implementation omitted 
  } 
}
```

## @RequestHeader

```java
@RequestMapping("/test") 
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding, 
                              @RequestHeader("Keep-Alive")long keepAlive)  { 
 
}
```



## @CookieValue

```java
@RequestMapping("/test")  
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie)  {  

}
```



## @RequestBody和@ResponseBody（略）



## @PropertySource的使用

如果配置文件中内容比较多，通常情况下会将配置文件进行切分，拆成多个。默认情况下spring只会读取几个特定命名的配置文件（application.properties或者application.yml)，如果需要制定配置文件的路径就需要用到该注解了



### 与@value配合使用



demo.properties

```
demo.name=huang
demo.sex=1
demo.type=demo
```

```java
@Component
@PropertySource(value = {"demo/props/demo.properties"})
public class ReadByPropertySourceAndValue {

    @Value("${demo.name}")
    private String name;

    @Value("${demo.sex}")
    private int sex;

    @Value("${demo.type}")
    private String type;
  	
  	// getset略
}
```



### 与@ConfigurationProperties配合使用

这是spring boot中的注解

```java
@Component
@PropertySource(value = {"demo/props/demo.properties"})
@ConfigurationProperties(prefix = "demo")
public class ReadByPropertySourceAndConfProperties {

    private String name;

    private int sex;

    private String type;
  
  	// getset略
}
```



## @Conditional

@Conditional是Spring4新提供的注解，它的作用是按照一定的条件进行判断，满足条件给容器注册bean

### 源码

```java
//此注解可以标注在类和方法上
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME) 
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```

从代码中可以看到，需要传入一个Class数组，并且需要继承Condition接口：

```java
public interface Condition {
    boolean matches(ConditionContext var1, AnnotatedTypeMetadata var2);
}
```

Condition是个接口，需要实现matches方法，返回true则注入bean，false则不注入

### 示例



#### person类

```java
public class Person {
 
    private String name;
    private Integer age;
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public Integer getAge() {
        return age;
    }
 
    public void setAge(Integer age) {
        this.age = age;
    }
 
    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
 
    @Override
    public String toString() {
        return "Person{" + "name='" + name + '\'' + ", age=" + age + '}';
    }
}
```

#### BeanConfig类

```java
@Configuration
public class BeanConfig {
 
    @Bean(name = "bill")
    public Person person1(){
        return new Person("Bill Gates",62);
    }
 
    @Bean(name = "linus")
    public Person person2(){
        return new Person("Linus",48);
    }
}
```

#### 测试类进行验证这两个Bean是否注入成功

```java
public class ConditionalTest {
 
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(BeanConfig.class);
 
    @Test
    public void test1(){
        Map<String, Person> map = applicationContext.getBeansOfType(Person.class);
        System.out.println(map);
    }
}
```

#### 输出结果

```
{linus=Person{name='Linus', age=48}, bill=Person{name='Bill Gates', age=62}}
```

发现有两个bean都被注入了



#### 开始使用@Conditional注解

如果想**根据当前操作系统来注入Person实例**，windows下注入bill，linux下注入linus，那么就需要使用该注解了。

##### WindowsCondition

```java
public class WindowsCondition implements Condition {
 
    /**
     * @param conditionContext:判断条件能使用的上下文环境
     * @param annotatedTypeMetadata:注解所在位置的注释信息
     * */
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        //获取ioc使用的beanFactory
        ConfigurableListableBeanFactory beanFactory = conditionContext.getBeanFactory();
        //获取类加载器
        ClassLoader classLoader = conditionContext.getClassLoader();
        //获取当前环境信息
        Environment environment = conditionContext.getEnvironment();
        //获取bean定义的注册类
        BeanDefinitionRegistry registry = conditionContext.getRegistry();
 
        //获得当前系统名
        String property = environment.getProperty("os.name");
        //包含Windows则说明是windows系统，返回true
        if (property.contains("Windows")){
            return true;
        }
        return false;
    }
}
```

##### LinuxCondition

```java
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class LinuxCondition implements Condition {
 
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
 
        Environment environment = conditionContext.getEnvironment();
 
        String property = environment.getProperty("os.name");
        if (property.contains("Linux")){
            return true;
        }
        return false;
    }
}
```

##### 修改BeanConfig

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BeanConfig {
 
    @Bean(name = "bill")
    @Conditional(WindowsCondition.class)
    public Person person1(){
        return new Person("Bill Gates",62);
    }

    @Conditional(LinuxCondition.class)
    @Bean(name = "linus")
    public Person person2(){
        return new Person("Linus",48);
    }
}
```

##### 再度测试结果

我本地机器是 mac， 可以在jvm 启动参数（设置参数方式如下图） 加上 Linux 或者 Windows进行切换做测试，这里以Linux做测试

```
{linus=Person{name='Linus', age=48}}
```

只注册了一个bean

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191114173515.png)



-Dos.name=Linux



# Spring 4.x 和 Spring 5.x注解时代

主要是 对@Conditional 注解进行了一些扩展 在 该注解基础上加入了 一些扩展，列举了一些如下

| 条件化注解                      | 配置生效条件                                         |
| :------------------------------ | :--------------------------------------------------- |
| @ConditionalOnBean              | 配置了某个特定bean                                   |
| @ConditionalOnMissingBean       | 没有配置特定的bean                                   |
| @ConditionalOnClass             | Classpath里有指定的类                                |
| @ConditionalOnMissingClass      | Classpath里没有指定的类                              |
| @ConditionalOnExpression        | 给定的Spring Expression Language表达式计算结果为true |
| @ConditionalOnJava              | Java的版本匹配特定指或者一个范围值                   |
| @ConditionalOnProperty          | 指定的配置属性要有一个明确的值                       |
| @ConditionalOnResource          | Classpath里有指定的资源                              |
| @ConditionalOnWebApplication    | 这是一个Web应用程序                                  |
| @ConditionalOnNotWebApplication | 这不是一个Web应用程序                                |



引入了一些注解例如：@RestController、@GetMapping、@PutMapping等



# 扩展：自动配置(重点)

在上面例子中讲解 @Import注解的时候 所有的类都在一个包下。这里做一下调整

## annotion包下的类

### 先看测试类

```java
package com.jx.learn.annotation;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.jx.learn.annotation");
     
        Teacher teacher = applicationContext.getBean(Teacher.class);
        System.out.println(teacher.getName());
    }
}
```

代码很清晰，在com.jx.learn.annotation 包下的扫描，然后获取Teacher的bean对象，打印名称

### Teacher类、Student类

```java
package com.jx.learn.annotation;
import lombok.Getter;
import lombok.Setter;
@Getter
@Setter
public class Teacher {
    private String name;
    private Student student;
}



package com.jx.learn.annotation;
import lombok.Getter;
import lombok.Setter;
@Getter
@Setter
public class Student {
    private Integer age;
}
```

没什么实质内容，继续看配置类

### 查看配置类

```java
package com.jx.learn.annotation;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyConfig2 {

    @Bean
    public Teacher getTeacher(Student student, String name){
        System.out.println(student.getAge());
        Teacher teacher = new Teacher();
        teacher.setName(name);
        teacher.setStudent(student);
        return teacher;
    }
}
```

该类也在 com.jx.learn.annotation 下，所以会被扫描到，意图很简单，返回一个Teacher 的 bean对象。但是返回的时候注入了 name 和 student两个bean。但是 name和student两个bean 在另外一个包下



## annotion2包下的类

```java
package com.jx.learn.annotation2;


import com.jx.learn.annotation.Student;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyConfig {

    @Bean
    public Student getStudent(){
        Student student = new Student();
        student.setAge(10);
        return student;
    }

    @Bean
    public String setName(){
        return "jack";
    }
}
```

## 测试

```
Error creating bean with name 'getTeacher' defined in com.jx.learn.annotation.MyConfig2: Unsatisfied dependency expressed through method 'getTeacher' parameter 0; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.jx.learn.annotation.Student' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {}
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:749)
```

错误描述很清晰，创建Teacher bean的时候 出错，因为依赖的 Student bean不存在



## 解决方式

在annotaton包下的MyConfig2 加上注解 @import 引入  Myconfig即可



```java
package com.jx.learn.annotation;

import com.jx.learn.annotation2.MyConfig;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Import(MyConfig.class)
public class MyConfig2 {

    @Bean
    public Teacher getTeacher(Student student, String name){
        System.out.println(student.getAge());
        Teacher teacher = new Teacher();
        teacher.setName(name);
        teacher.setStudent(student);
        return teacher;
    }
}
```



## 说明

这种模式在Spring Boot中被大量使用，实际上就是 @Enable模式。例如 @EnableMvc、@EnableEurekaServer



他们的共性就是引入的第三方包，路径当然就不在Spring Boot 配置的扫描范围内，那么 引入 一个 @Enablexxx注解，然后  @Enablexxx 注解里面又包含一个 @Import来引入配置。



以代码简单说明



### 启动类

```java
package com.jx
import com.hello.EnableHello

@Configuration
@EnableHello
public class BootStrap{
	public static void main(String[] args){
		// 什么包都不扫描
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext();
	}
}
```

### EnableHello

```java
package com.hello

@Import(HelloConfiguration.class)
public @interface EnableHello{
  
}
```

### HelloConfiguration

```java
package com.hello

@Configuration
public class HelloConfiguration{

}
```





## 结束

通过上面自动配置，讲解了 为什么EnableXXX 就能够使用了。  然后再结合  @Conditional注解以及扩展条件注解，就可以实现自动配置了



