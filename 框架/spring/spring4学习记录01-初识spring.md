
# 简介
spring是一个开源框架，spring可以做非常多的事情，但是spring的目标是致力于简化Java开发。 
为了降低spring开发的复杂性，spring采取了以下四个关键策略. 
- 基于pojo的轻量级和最小入侵性编程
- 通过依赖注入和面向接口实现松耦合
- 基于面向切面的惯例进行声明式编程
- 通过切面和模板减少样板式代码
# 说明spring核心
## IOC(控制反转)
场景：人读书

那么定义两个类 人 和 书

```
public class People {
    public String name;
    private int age;
    private Book book;
    public void readBook(){
        System.out.println(book.getName());
    }
    public Book getBook () {
        return book;
    }
    public void setBook(Book book) {
        this.book = book;
    }
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

```
public class Book {
    private String name;
    private String author;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getAuthor() {
        return author;
    }
    public void setAuthor(String author) {
        this.author = author;
    }
}
```
如果直接执行下面的代码就会抛出 空指针异常 .

```
	 public static void main(String[] args) {
        People people = new People();
        people.readBook();
    }
```
必须改成

```
	public static void main(String[] args) {
        People people = new People();
        Book book = new Book();
        people.setBook(book);
        people.readBook();
    }
```
这里就需要对people对象的构建，book对象的构建全部由程序来完成. 
而交给spring管理呢，就会帮助我们省去手动new一个的对象的过程，由spring帮助我们自动生成这个对象，而免去手动构造一个对象，手动销毁一个对象都交由spring控制，这就叫做**控制反转**.

交给spring管理的代码片段

```
public class Main {

    private static ApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");

    public static void main(String[] args) {
        People people = (People) context.getBean("people");
        people.readBook();
    }

}
```
### DI(依赖注入)
上面的例子中Book就是通过DI注入到people中，people不用关注book从哪里来，只需要调用读书的方法就好,book是spring通过反射生成交给people的.

## AOP(面向切面)
场景 人跑步

定义两个类，一个run，一个通知类advices

```
/**
 * 被代理的目标类
 */
public class Run{
    public void run(String name , int km){
        System.out.println(name + " : run" + km);
    }
}
```

```
import org.aspectj.lang.JoinPoint;

/**
 * 通知类，横切逻辑
 *
 */
public class Advices {

    public void before(JoinPoint jp){
        System.out.println("----------跑步之前注意热身，别受伤----------");
    }

    public void after(JoinPoint jp){
        System.out.println("----------跑步完了放松肌肉----------");
    }
}
```
想要在跑步之前提醒跑步者注意热身，跑步完之后提醒跑步者进行放松.

```
<bean id="run" class="com.jx.spring01.Run"/>
    <bean id="advices" class="com.jx.spring01.Advices"/>

    <!-- aop配置 -->
    <aop:config proxy-target-class="true">
        <!--切面 -->
        <aop:aspect ref="advices">
            <!-- 切点 -->
            <aop:pointcut expression="execution(* com.jx.spring01.Run.*(..))" id="pointcut1"/>
            <!--连接通知方法与切点 -->
            <aop:before method="before" pointcut-ref="pointcut1"/>
            <aop:after method="after" pointcut-ref="pointcut1"/>
        </aop:aspect>
    </aop:config>
```

调用测试类

```
private static ApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");

    public static void main(String[] args) {
        Run run = (Run) context.getBean("run");
        run.run("张三", 10);
    }
```
打印结果

```
----------跑步之前注意热身，别受伤----------
张三 : run10
----------跑步完了放松肌肉----------
```
本章内容只是入门简单说明spring的核心ioc和aop的概念。具体用法后面会提到.