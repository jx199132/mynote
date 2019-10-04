
# 前置消息，后置消息，环绕消息，参数传递

```
package com.jx.spring02;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;

public class Calc {
	
	public void before(JoinPoint jp){
		System.out.println("----------跑步之前注意热身，别受伤----------");
    }
	
	public void after(JoinPoint jp){
        System.out.println("----------跑步完了放松肌肉----------");
    }
	
	public void around(ProceedingJoinPoint jp) throws Throwable{
		System.out.println("开始时间：" + System.currentTimeMillis());
		jp.proceed();
		System.out.println("结束跑步时间:" + System.currentTimeMillis());
	}
	
	public void params(int year , int total){
		System.out.println("year : " + year + " , total : " + total);
	}
}
```

```
package com.jx.spring02;

public class Runner {
	private String name;
	private int km;
	public int getKm() {
		return km;
	}
	public void setKm(int km) {
		this.km = km;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public void run(){
		System.out.println(this.name + " 跑了" + this.getKm() + "公里步 ");
	}

	public void sayMessage(int year , int total){
		System.out.println("我叫" + this.name + "我从" + year + "年开始跑步，至今为止跑了" + total + "公里");
	}
}

```

```
package com.jx.spring02;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main1 {
	
	private static ApplicationContext context;
	static{
		context = new ClassPathXmlApplicationContext("spring-02.xml");
	}
	
	public static void main(String[] args) {
		Runner run = (Runner) context.getBean("runner");
		run.setKm(10);
		run.setName("张三");
		run.run();
		run.sayMessage(2000 , 3000);
	}
}
```

```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context" 
       xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
          http://www.springframework.org/schema/aop
          http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
          http://www.springframework.org/schema/context    
      http://www.springframework.org/schema/context/spring-context-4.3.xsd    
          ">

  
  <bean id="runner" class="com.jx.spring02.Runner"></bean>
  <bean id="calc" class="com.jx.spring02.Calc"/> 
  <!-- 定义配置 -->
  <aop:config proxy-target-class="true">
  	<!-- 定义一个切面，来提醒runner在跑步之前进行热身，跑步之后进行放松  ,通过前后通知的形式 , 并且记录开始跑步时间和结束跑步时间通过环绕通知的形式-->
  	<aop:aspect ref="calc">
  		<aop:pointcut expression="execution(* com.jx.spring02.Runner.run(..))" id="pointcut1"/>
  		<!--连接通知方法与切点 -->
        <aop:before method="before" pointcut-ref="pointcut1"/>
        <aop:after method="after" pointcut-ref="pointcut1"/>
        <aop:around method="around" pointcut-ref="pointcut1"/>
        
        <aop:pointcut expression="execution(* com.jx.spring02.Runner.sayMessage(..)) and args(year,total)" id="pointcut2"/>
        <aop:before method="params" pointcut-ref="pointcut2" arg-names="year,total"/>
  </aop:config>
  
</beans>
```

# 引入

定义一个类和接口没有任何方法与字段  来通过spring aop实现 可以调用其他接口的方法
```
package com.jx.spring03;

public interface ArithmeticCalculator {

}
```

```
public class ArithmeticCalculatorImpl implements ArithmeticCalculator {
	
} 
```
接口一获取最大的值
```
package com.jx.spring03;
public interface MaxCalculator {  
    double max(double a,double b);  
} 
```

```
package com.jx.spring03;
public class MaxCalculatorImpl implements MaxCalculator {  
  
    @Override  
    public double max(double a, double b) {  
        double result = a<=b?b:a;  
        System.out.println("max("+a+","+b+")="+result);  
        return result;  
    }  
  
} 
```
接口二获取最小的值
```
package com.jx.spring03;
public interface MinCalculator {  
  
    double min(double a,double b);  
}
```

```
package com.jx.spring03;
public class MinCalculatorImpl implements MinCalculator {  
  
    @Override  
    public double min(double a, double b) {  
        double result = a<=b?a:b;  
        System.out.println("min("+a+","+b+")="+result);  
        return result;  
    }  
  
}
```
定义一个切面
```
package com.jx.spring03;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.DeclareParents;

/**
 * 定义一个切面
 */
@Aspect  
public class CalculatorIntroduction {

    @DeclareParents(value="com.jx.spring03.ArithmeticCalculatorImpl",
            defaultImpl=MaxCalculatorImpl.class)
    public MaxCalculator maxCalculator;

    @DeclareParents(value="com.jx.spring03.ArithmeticCalculatorImpl",
            defaultImpl=MinCalculatorImpl.class)
    public MinCalculator minCalculator;
}
```
测试类
```
package com.jx.spring03;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main2 {
	
    private static ApplicationContext context;  
    
    static{
            context = new  ClassPathXmlApplicationContext("spring-03.xml");  
    }  
    
    public static void main(String[] args) {  
        ArithmeticCalculator ac = (ArithmeticCalculator)context.getBean("arithmeticCalculator");  

        MinCalculator min = (MinCalculator)ac;
        min.min(2.3,4.5);

        MaxCalculator max = (MaxCalculator)ac;
        max.max(3, 5);
    }  
} 
```
spring配置文件
```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context" 
       xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
          http://www.springframework.org/schema/aop
          http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
          http://www.springframework.org/schema/context    
      http://www.springframework.org/schema/context/spring-context-4.3.xsd    
          ">

  <!-- 开启注解 -->
  <context:annotation-config/>
  <aop:aspectj-autoproxy/>
  
  <bean class="com.jx.spring03.CalculatorIntroduction"></bean>  
  <bean id="arithmeticCalculator" class="com.jx.spring03.ArithmeticCalculatorImpl" />  
</beans>
```


# 使用场景
- 日志记录
- 控制层异常处理
- 动态数据库切换