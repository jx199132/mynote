# ApplicationContextInitializer介绍

- 用于在spring容器刷新之前初始化Spring ConfigurableApplicationContext的回调接口
- 通常用于需要对应用程序上下文进行编程初始化的web应用程序中。例如，根据上下文环境注册属性源或激活配置文件等
- 可排序的（实现Ordered接口，或者添加@Order注解）





# 三种实现方式



## 直接添加

- 新建一个类继承与ApplicationContextInitializer

```java
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;

public class MyApplicationContextInitializer implements ApplicationContextInitializer {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        System.out.println("-----MyApplicationContextInitializer initialize-----");
    }
}
```

- 在springboot启动类，实例化完毕后添加进去

  ```java
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  
  @SpringBootApplication
  public class ApplicationContextInitializerApplication {
  
      public static void main(String[] args) {
          SpringApplication application = new SpringApplication(ApplicationContextInitializerApplication.class);
          application.addInitializers(new MyApplicationContextInitializer());
          application.run(args);
      }
  
  }
  ```

- 查看结果（可以看到会有输出）

  ```
   /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
  ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
   \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
    '  |____| .__|_| |_|_| |_\__, | / / / /
   =========|_|==============|___/=/_/_/_/
   :: Spring Boot ::        (v2.2.0.RELEASE)
  
  -----MyApplicationContextInitializer initialize-----
  2019-10-24 13:14:29.649  INFO 3245 --- [           main] ApplicationContextInitializerApplication : Starting ApplicationContextInitializerApplication on no1deMacBook-Pro.local with PID 3245 (/Users/no1/work/idea/springboot/application_context_initializer/target/classes started by no1 in /Users/no1/work/idea/springboot)
  2019-10-24 13:14:29.652  INFO 3245 --- [           main] ApplicationContextInitializerApplication : No active profile set, falling back to default profiles: default
  2019-10-24 13:14:30.080  INFO 3245 --- [           main] ApplicationContextInitializerApplication : Started ApplicationContextInitializerApplication in 0.68 seconds (JVM running for 1.194)
  
  ```

  ### 配置文件中配置

在 application.properties 中加上

```
context.initializer.classes=com.jx.learn.sprngboot.application_context_initializer.MyApplicationContextInitializer
```

### SpringBoot的SPI扩展---META-INF/spring.factories中配置

```
org.springframework.context.ApplicationContextInitializer=com.jx.learn.sprngboot.application_context_initializer.MyApplicationContextInitializer
```

