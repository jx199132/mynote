# 开篇

在上一篇中讲到了spring 的 自动配置，这一篇谈谈spring boot在自动配置中是如何做的



# 浅析springboot自动配置

spring boot提供了大量的自动配置，用来减少用户（框架使用者）来配置，下面跟着源码进行解读，从入口启动类开始

```java
@SpringBootApplication
public class Boot01Application {
    public static void main(String[] args) {
        SpringApplication.run(Boot01Application.class, args);
    }
}
```

入口程序中会有一个 注解 @SpringBootApplication，下面看看注解的源码

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

会有@EnableAutoConfiguration 注解，继续跟下去看源码

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
}
```

在上一篇中就详细介绍了spring的自动配置就是用的@import注解，这里@import引入了AutoConfigurationImportSelector.class，继续看 改类的源码

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
		ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
}
```

可以发现AutoConfigurationImportSelector 实现了DeferredImportSelector接口，而DeferredImportSelector接口又继承ImportSelector， 下面继续看看该接口源码

```java
public interface ImportSelector {
	String[] selectImports(AnnotationMetadata importingClassMetadata);
}
```

ImportSelect的作用是收集需要导入的配置类，只有一个方法，该方法返回需要配置的类的名称，那么回到AutoConfigurationImportSelector类，看看AutoConfigurationImportSelector#selectImports源码

```java
@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
    // 1
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
    
    // 2
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
				annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```

1. AutoConfigurationMetadataLoader#loadMetadata(this.beanClassLoader) 看源码可以发现读取的是配置文件 "META-INF/spring-autoconfigure-metadata.properties"，**该文件是条件注入的一些信息，来描述什么条件下加载哪些bean**，部分内容如下

   ```properties
   org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration.ConditionalOnClass=org.springframework.data.jdbc.repository.config.JdbcConfiguration,org.springframework.jdbc.core.namedparam.NamedParameterJdbcOperations
   org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration.Configuration=
   org.springframework.boot.autoconfigure.jms.activemq.ActiveMQXAConnectionFactoryConfiguration.ConditionalOnClass=javax.transaction.TransactionManager
   org.springframework.boot.autoconfigure.kafka.KafkaAnnotationDrivenConfiguration.ConditionalOnClass=org.springframework.kafka.annotation.EnableKafka
   org.springframework.boot.autoconfigure.hazelcast.HazelcastClientConfiguration.ConditionalOnClass=com.hazelcast.client.HazelcastClient
   org.springframework.boot.autoconfigure.jms.artemis.ArtemisXAConnectionFactoryConfiguration=
   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration.Configuration=
   org.springframework.boot.autoconfigure.data.mongo.MongoDbFactoryDependentConfiguration=
   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration=
   org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration.AutoConfigureAfter=org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration
   org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration.Configuration=
   ```

2. 继续看getAutoConfigurationEntry的代码，忽略了其他代码，只保留了需要看的部分

   ```java
   protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
   			AnnotationMetadata annotationMetadata) {
   		// 其他代码忽略
   		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
   		return new AutoConfigurationEntry(configurations, exclusions);
   	}
   ```

   跟踪getCandidateConfigurations的代码

   ```java
   protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
   		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
   				getBeanClassLoader());
   		return configurations;
   	}
   ```

   那么继续跟踪SpringFactoriesLoader.loadFactoryNames

   ```java
   public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
   		String factoryClassName = factoryClass.getName();
   		return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
   	}
   ```

   跟踪loadSpringFactories

   ```java
   private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
   			Enumeration<URL> urls = (classLoader != null ?
   					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
   					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
   }		
   ```

   发现ClassLoader加载了资源文件，这个常量FACTORIES_RESOURCE_LOCATION的值为"META-INF/spring.factories，那么查看一下这个文件，部分内容如下

   ![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191116222010.png)

   **该文件就是springboot提供的大量自动配置的一些配置类**，他们的配置定义如下图

   <img src="https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191116222356.png" style="zoom:33%;" />

   # 总结

   spring中提供了自动配置的功能，但是配置文件需要自己写。而spring boot在自动配置功能的基础上 提供了大量的默认配置类（@Configuration）来 对自动配置功能进行补充

   

   那么大量的默认配置存在哪个包下面呢就由 **META-INF/spring.factories**中进行描述

   

   大量的配置类，当然不可能每一个都进行加载，那么就需要 条件注入（@Conditional）进行控制了，大部分配置类都做了控制，还有一个地方就是 **spring-autoconfigure-metadata.properties** 配置文件



