# 共同点和区别



## 共同点

都是接口



## 区别

**BeanFactory** 以Factory结尾，表示它是一个工厂类，用于管理Bean的一个工厂在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的



**FactoryBean**这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean。它的实现与设计模式中的工厂模式和修饰器模式类似。一句话就是  它是一个  **生成或者修饰Bean的Bean**



# 使用浅析



## BeanFactory

BeanFactory定义了IOC容器的最基本形式，并提供了IOC容器应遵守的的最基本的接口，也就是Spring IOC所遵守的最底层和最基本的编程规范。



它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖



在Spring代码中，BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等，都是附加了某种功能的实现



- BeanFactory–>Spring IoC容器顶级接口,定义了对单个bean的获取,对bean的作用域判断,获取bean类型,获取bean别名的功能

  

- ListableBeanFactory–>扩展了BeanFactory接口,并提供了对bean的枚举能力

  

- HierarchicalBeanFactory–>扩展了BeanFactory接口,并提供了访问父容器的能力

  

- AutowireCapableBeanFactory–>扩展了BeanFactory接口,并提供了自动装配能力

  

- ConfigurableBeanFactory–>扩展了HierarchicalBeanFactory,并提供了对容器的配置能力

  

- ConfigurableListableBeanFactory–>扩展了ListableBeanFactory, AutowireCapableBeanFactory, 

  

- ConfigurableBeanFactory接口,并提供了忽略依赖,自动装配判断,冻结bean的定义,枚举所有bean名称的功能
  

```java
package org.springframework.beans.factory;

import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;

public interface BeanFactory {
    String FACTORY_BEAN_PREFIX = "&";

    Object getBean(String var1) throws BeansException;

    <T> T getBean(String var1, Class<T> var2) throws BeansException;

    <T> T getBean(Class<T> var1) throws BeansException;

    Object getBean(String var1, Object... var2) throws BeansException;

    <T> T getBean(Class<T> var1, Object... var2) throws BeansException;

    boolean containsBean(String var1);

    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;

    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, ResolvableType var2) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, Class<?> var2) throws NoSuchBeanDefinitionException;

    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;

    String[] getAliases(String var1);
}
```





## FactoryBean

一般情况下，Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置方式的灵活性和复杂度限制，这时采用编码的方式可能会更加容易



Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑



FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现，第三方框架要继承进Spring，往往就是通过实现FactoryBean来集成的。比如MyBatis的SqlSessionFactoryBean、RedisRepositoryFactoryBean、EhCacheManagerFactoryBean等等



```java
package org.springframework.beans.factory;

public interface FactoryBean<T> {
  	// 返回由FactoryBean创建的Bean实例，如果isSingleton()返回true，则该实例会放到Spring容器中单实例缓存池中
    T getObject() throws Exception;
	
  	//	返回FactoryBean创建的Bean类型
    Class<?> getObjectType();
  
		//	返回由FactoryBean创建的Bean实例的作用域是否singleton
    boolean isSingleton();
}
```



当配置文件中<bean>的class属性配置的实现类是FactoryBean时，通过getBean()方法返回的不是FactoryBean本身。而是FactoryBean#getObject()方法所返回的对象，相当于FactoryBean#getObject()代理了getBean()方法



### 示例1

如果使用传统方式配置下面Car的<bean>时，Car的每个属性分别对应一个<property>元素标签

```java
public class Car {
    private int maxSpeed;
    private String brand;
    private double price;

    public int getMaxSpeed() {
        return this.maxSpeed;
    }

    public void setMaxSpeed(int maxSpeed) {
        this.maxSpeed = maxSpeed;
    }

    public String getBrand() {
        return this.brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public double getPrice() {
        return this.price;
    }

    public void setPrice(double price) {
        this.price = price;
    }
}
```



如果用FactoryBean的方式实现就灵活点，下例通过逗号分割符的方式一次性的为Car的所有属性指定配置值： 

```java
import org.springframework.beans.factory.FactoryBean;

public class CarFactoryBean implements FactoryBean<Car> {
    private String carInfo;

    public Car getObject() throws Exception {
        Car car = new Car();
        String[] infos = carInfo.split(",");
        car.setBrand(infos[0]);
        car.setMaxSpeed(Integer.valueOf(infos[1]));
        car.setPrice(Double.valueOf(infos[2]));
        return car;
    }

    public Class<Car> getObjectType() {
        return Car.class;
    }

    public boolean isSingleton() {
        return false;
    }

    public String getCarInfo() {
        return this.carInfo;
    }

    // 接受逗号分割符设置属性信息  
    public void setCarInfo(String carInfo) {
        this.carInfo = carInfo;
    }
}
```



有了这个CarFactoryBean后，就可以在配置文件中使用下面这种自定义的配置方式配置CarBean了

```xml
<bean id="car"class="com.baobaotao.factorybean.CarFactoryBean">
  <prototype name="carInfo" value="法拉利,400,2000000"/>
</bean>
```

当调用getBean("car")时，Spring通过反射机制发现CarFactoryBean实现了FactoryBean的接口，这时Spring容器就调用接口方法CarFactoryBean#getObject()方法返回



如果希望获取CarFactoryBean的实例，则需要在使用getBean(beanName)方法时在beanName前显示的加上"&"前缀：如getBean("&car");





### 示例2

在通过一个示例，用来代理一个对象。从而可以很方便的在对象前后都做出对应的操作

```java
public class MyFactoryBean implements FactoryBean<Object> {

    private static final Logger logger = LoggerFactory.getLogger(MyFactoryBean.class);
    private Class<?> interfaceClazz; //实现的接口的全类名
    private Object target; //该接口的实现类
    private Object proxyObj;

    public MyFactoryBean(Class<?> interfaceClazz, Object target) {
        this.interfaceClazz = interfaceClazz;
        this.target = target;
        this.proxyObj = Proxy.newProxyInstance(
                this.getClass().getClassLoader(),
                new Class[]{interfaceClazz}, //默认必须实现这个接口
                (proxy, method, args) -> {
                    logger.debug("invoke method......" + method.getName());
                    logger.debug("invoke method before......" + System.currentTimeMillis());
                    Object result = method.invoke(target, args);
                    logger.debug("invoke method after......" + System.currentTimeMillis());

                    return result;
                });
    }

    @Override
    public Object getObject() {
        return proxyObj; //返回这个代理对象 而不是new直接new出来的对象
    }

    @Override
    public Class<?> getObjectType() {
        return proxyObj == null ? Object.class : proxyObj.getClass();
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```



调用代码

```java
     public static void main(String[] args) {
        MyFactoryBean factoryBean = new MyFactoryBean(UserService.class, new UserServiceImpl());

        UserService userService = (UserService) factoryBean.getObject();
        System.out.println(userService.sayHello());
    }

    private interface UserService {
        String sayHello();
    }

    private static class UserServiceImpl implements MyFactoryBean.UserService {

        @Override
        public String sayHello() {
            return "hello world";
        }
    }

```

控制台输出

```
16:17:05.330 [main] DEBUG com.fsx.single.temp.MyFactoryBean - invoke method......sayHello
16:17:05.334 [main] DEBUG com.fsx.single.temp.MyFactoryBean - invoke method before......1545121025334
16:17:05.334 [main] DEBUG com.fsx.single.temp.MyFactoryBean - invoke method after......1545121025334
hello world
```

