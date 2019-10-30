# 什么是事务

事务是逻辑上的一组操作，要么都执行，要么都不执行



# 事物的特性（ACID）

- 原子性： 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
- 一致性： 执行事务前后，数据保持一致；
- 隔离性： 并发访问数据库时，一个用户的事物不被其他事物所干扰，各并发事务之间数据库是独立的；
- 持久性: 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响



# Spring事务管理接口介绍

spring事务管理相关的接口

- **PlatformTransactionManager：** （平台）事务管理器
- **TransactionDefinition：** 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)
- **TransactionStatus：** 事务运行状态



**所谓事务管理，其实就是“按照给定的事务规则来执行提交或者回滚操作”**



## PlatformTransactionManager接口介绍

Spring并不直接管理事务，而是提供了多种事务管理器 ，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 Spring事务管理器的接口是： org.springframework.transaction.PlatformTransactionManager ，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了



接口定义源码如下：

```java
Public interface PlatformTransactionManager()...{  
    // 根据指定的传播行为，返回当前活动的事务或创建一个新事务
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
    // 使用事务目前的状态提交事务
    Void commit(TransactionStatus status) throws TransactionException;  
    // 对执行的事务进行回滚
    Void rollback(TransactionStatus status) throws TransactionException;  
} 
```

比如我们在使用JDBC或者Mybatis进行数据持久化操作时,我们的xml配置通常如下：

```xml
    <!-- 事务管理器 -->
    <bean id="transactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 数据源 -->
        <property name="dataSource" ref="dataSource" />
    </bean>
```



## TransactionDefinition接口介绍

事务管理器接口 PlatformTransactionManager 通过 getTransaction(TransactionDefinition definition) 方法来得到一个事务，这个方法里面的参数是 TransactionDefinition类 ，这个类就定义了一些基本的事务属性

事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。事务属性包含了5个方面

```java
public interface TransactionDefinition {
    // 返回事务的传播行为
    int getPropagationBehavior(); 
    
    // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getIsolationLevel(); 
    
    // 返回事务的名字
    String getName()；
    
    // 返回事务必须在多少秒内完成
    int getTimeout();  
    
    // 返回是否优化为只读事务。
    boolean isReadOnly();
} 
```



### 事务隔离级别

- TransactionDefinition.ISOLATION_DEFAULT: 使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED: 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
- TransactionDefinition.ISOLATION_READ_COMMITTED: 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
- TransactionDefinition.ISOLATION_REPEATABLE_READ: 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
- TransactionDefinition.ISOLATION_SERIALIZABLE: 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别



### 事务传播机制

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：



#### 支持当前事务的情况

- TransactionDefinition.PROPAGATION_REQUIRED： 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

  

- TransactionDefinition.PROPAGATION_SUPPORTS： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

  

- TransactionDefinition.PROPAGATION_MANDATORY： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常
  



#### 不支持当前事务的情况

- TransactionDefinition.PROPAGATION_REQUIRES_NEW： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_NOT_SUPPORTED： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_NEVER： 以非事务方式运行，如果当前存在事务，则抛出异常。

#### 其他情况

- TransactionDefinition.PROPAGATION_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED



## TransactionStatus接口介绍

TransactionStatus接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息.

PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）

```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事物
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
} 
```



## Spring事务不生效的原因大解读

### 一些原因

**原因一：**是否是数据库引擎设置不对造成的。比如我们最常用的mysql，引擎MyISAM，是不支持事务操作的。需要改成InnoDB才能支持



**原因二：**入口的方法必须是public，否则事务不起作用（这一点由Spring的AOP特性决定的，理论上而言，不public也能切入，但spring可能是觉得private自己用的方法，应该自己控制，不应该用事务切进去吧）。另外private 方法, final 方法 和 static 方法不能添加事务，加了也不生效



**原因三：**Spring的事务管理发现异常，进行回滚



**原因五：**请确认你的类是否被代理了（因为spring的事务实现原理为AOP，只有通过代理对象调用方法才能被拦截，事务才能生效）



**原因六：**请确保你的业务和事务入口在同一个线程里，否则事务也是不生效的，比如下面代码事务不生效：

```java
@Transactional
@Override
public void save(User user1, User user2) {
    new Thread(() -> {
          saveError(user1, user2);
          System.out.println(1 / 0);
    }).start();
}

```



### 一些示例

图一：事务不生效：.@Transactional的事务开启 ，或者是基于接口的 或者是基于类的代理被创建。所以在同一个类中**一个无事务的方法调用另一个有事务的方法，事务是不会起作用的**

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191028160054.png)



图二：事务生效

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191030161308.png)

图三：事务生效

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191030161338.png)



图四：事务生效

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191030161400.png)



图五：事务生效

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191030161420.png)



图六：事务不生效（准确的说这叫没有事务）

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191030161442.png)



图七：事务生效

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191030161457.png)





## Spring事务嵌套引发的血案---Transaction rolled back because it has been marked as rollback-only



这种写法是我们最为普通的写法，显然是可以回滚的

```java
@Transactional
@Override
public boolean create(User user) {
    int i = userMapper.insert(user);
    personService.addPerson(user);
    return i == 1;
}

//下面是personService的addPerson方法，也是有事务的
@Transactional
@Override
 public boolean addPerson(User user) {
     System.out.println(1 / 0);
     return false;
 }

```



那么换一种写法

```java
 @Transactional
 @Override
  public boolean create(User user) {
      int i = userMapper.insert(user);
      try {
          personService.addPerson(user);
      } catch (Exception e) {
          System.out.println("不断程序，用来输出日志~");
      }
      return i == 1;
  }

```

把别的service方法try住，不希望它阻断我们的程序继续执行。表面上看合乎情理没毛病，但是

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191028160634.png)

