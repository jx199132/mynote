
# 源码解析spring boot启动流程


## 入口
```
@SpringBootApplication
public class RunByJarApplication {

    public static void main(String[] args) {
        SpringApplication.run(RunByJarApplication.class, args);
    }

}
```
- 从main方法中执行了SpringApplication.run方法
- 传入的参数是 当前类名，参数


## 进入 SpringApplication.run方法

首先调用的代码如下
```
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
		return new SpringApplication(primarySources).run(args);
	}
```
这里代码分为两段：
- 第一段是实例化一个 SpringApplication，传入参数是类名
- 第二段是实例化完毕之后执行run方法，参入参数为 args

### SpringApplication 实例化的过程

```
@SuppressWarnings({"unchecked", "rawtypes"})
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    //推断应用类型，后面会根据类型初始化对应的环境 ##1.1
    this.webApplicationType = deduceWebApplicationType();
    //读取classpath下 META-INF/spring.factories中已配置的 ApplicationContextInitializer，生成实例（没有进行初始化工作） ##1.2
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    //同上，只不过类型是 ApplicationListener ##1.3
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //根据调用栈，推断出 main 方法的类名
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

### 调用run方法的过程

```
/**
 * Run the Spring application, creating and refreshing a new
 * {@link ApplicationContext}.
 *
 * @param args the application arguments (usually passed from a Java main method)
 * @return a running {@link ApplicationContext}
 *
 * 运行spring应用，并刷新一个新的 ApplicationContext（Spring的上下文）
 * ConfigurableApplicationContext 是 ApplicationContext 接口的子接口。在 ApplicationContext
 * 基础上增加了配置上下文的工具。 ConfigurableApplicationContext是容器的高级接口
 */
public ConfigurableApplicationContext run(String... args) {
    //记录程序运行时间
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // ConfigurableApplicationContext Spring 的上下文
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    //从META-INF/spring.factories中获取监听器
    //1、获取并启动监听器
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
                args);
        //2、构造应用上下文环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        //处理需要忽略的Bean
        configureIgnoreBeanInfo(environment);
        //打印banner
        Banner printedBanner = printBanner(environment);
        ///3、初始化应用上下文
        context = createApplicationContext();
        //实例化SpringBootExceptionReporter.class，用来支持报告关于启动的错误
        exceptionReporters = getSpringFactoriesInstances(
                SpringBootExceptionReporter.class,
                new Class[]{ConfigurableApplicationContext.class}, context);
        //4、刷新应用上下文前的准备阶段
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        //5、刷新应用上下文
        refreshContext(context);
        //刷新应用上下文后的扩展接口
        afterRefresh(context, applicationArguments);
        //时间记录停止
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                    .logStarted(getApplicationLog(), stopWatch);
        }
        //发布容器启动完成事件
        listeners.started(context);
        callRunners(context, applicationArguments);
    } catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    } catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```

run过程中的重要步骤
1. 获取并启动监听器 (##2.1)
1. 构造应用上下文环境 (##2.2)
1. 初始化应用上下文 (##2.3)
1. 刷新应用上下文前的准备阶段 (##2.4)
1. 刷新应用上下文  (##2.5)
1. 刷新应用上下文后的扩展接口  (##2.6)



# 附录详解
## 1.1 推断应用类型

```
//常量值
private static final String[] WEB_ENVIRONMENT_CLASSES = {"javax.servlet.Servlet",
            "org.springframework.web.context.ConfigurableWebApplicationContext"};

private static final String REACTIVE_WEB_ENVIRONMENT_CLASS = "org.springframework."
        + "web.reactive.DispatcherHandler";

private static final String MVC_WEB_ENVIRONMENT_CLASS = "org.springframework."
        + "web.servlet.DispatcherServlet";

private static final String JERSEY_WEB_ENVIRONMENT_CLASS = "org.glassfish.jersey.server.ResourceConfig";

/**
 * 判断 应用的类型
 * NONE: 应用程序不是web应用，也不应该用web服务器去启动
 * SERVLET: 应用程序应作为基于servlet的web应用程序运行，并应启动嵌入式servlet web（tomcat）服务器。
 * REACTIVE: 应用程序应作为 reactive web应用程序运行，并应启动嵌入式 reactive web服务器。
 * @return
 */
private WebApplicationType deduceWebApplicationType() {
    //classpath下必须存在org.springframework.web.reactive.DispatcherHandler
    if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
            && !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)
            && !ClassUtils.isPresent(JERSEY_WEB_ENVIRONMENT_CLASS, null)) {
        return WebApplicationType.REACTIVE;
    }
    for (String className : WEB_ENVIRONMENT_CLASSES) {
        if (!ClassUtils.isPresent(className, null)) {
            return WebApplicationType.NONE;
        }
    }
    //classpath环境下存在javax.servlet.Servlet或者org.springframework.web.context.ConfigurableWebApplicationContext
    return WebApplicationType.SERVLET;
}
```

## 1.2 从classpath下META-INF/spring.factories中读取已配置的ApplicationContextInitializer（仅仅生成实例，不做初始化）

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[]{});
}

/**
 * 通过指定的classloader 从META-INF/spring.factories获取指定的Spring的工厂实例
 * @param type
 * @param parameterTypes
 * @param args
 * @param <T>
 * @return
 */
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
                                                      Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
    // Use names and ensure unique to protect against duplicates
    //通过指定的classLoader从 META-INF/spring.factories 的资源文件中，
    //读取 key 为 type.getName() 的 value
    Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    //创建Spring工厂实例
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
            classLoader, args, names);
    //对Spring工厂实例排序（org.springframework.core.annotation.Order注解指定的顺序）
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```
有一个方法很重要 loadFactoryNames() 这个方法很重要，这个方法是spring-core中提供的从META-INF/spring.factories中获取指定的类（key）的同一入口方法。

在这里，获取的是key为 org.springframework.context.ApplicationContextInitializer 的类

在这个环境中获取到的value为

```
"org.springframework.boot.diagnostics.FailureAnalyzer" -> {LinkedList@1682}  size = 17
"org.springframework.boot.env.EnvironmentPostProcessor" -> {LinkedList@1684}  size = 3
"org.springframework.boot.SpringApplicationRunListener" -> {LinkedList@1686}  size = 1
"org.springframework.context.ApplicationContextInitializer" -> {LinkedList@1688}  size = 6
"org.springframework.boot.env.PropertySourceLoader" -> {LinkedList@1690}  size = 2
"org.springframework.context.ApplicationListener" -> {LinkedList@1692}  size = 10
"org.springframework.boot.diagnostics.FailureAnalysisReporter" -> {LinkedList@1694}  size = 1
"org.springframework.boot.SpringBootExceptionReporter" -> {LinkedList@1696}  size = 1
"org.springframework.beans.BeanInfoFactory" -> {LinkedList@1698}  size = 1
"org.springframework.boot.autoconfigure.AutoConfigurationImportFilter" -> {LinkedList@1700}  size = 3
"org.springframework.boot.autoconfigure.AutoConfigurationImportListener" -> {LinkedList@1702}  size = 1
"org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvider" -> {LinkedList@1704}  size = 5
"org.springframework.boot.autoconfigure.EnableAutoConfiguration" -> {LinkedList@1706}  size = 117
```

### 分别来自不同的jar包，例如

```
2.1.9.RELEASE/spring-boot-2.1.9.RELEASE.jar
spring-boot-autoconfigure-2.1.9.RELEASE.jar
```

#### 查看spring-boot-2.1.9.RELEASE.jar!/META-INF/spring.factories

```
# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer
```
#### 查看spring-boot-autoconfigure-2.1.9.RELEASE.jar!/META-INF/spring.factories

```
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener
```

### ApplicationContextInitializer的理解和使用
看下一篇


## 1.3    同上，只不过类型变为 ApplicationListener
初始化classpath下 META-INF/spring.factories中已配置的 ApplicationListener。

　　 ApplicationListener 的加载过程和上面的 ApplicationContextInitializer 类的加载过程是一样的。至于 ApplicationListener 是spring的事件监听器，典型的观察者模式，通过 ApplicationEvent 类和 ApplicationListener 接口，可以实现对spring容器全生命周期的监听，也可以自定义监听事件


## 2.1 获取并启动监听器
事件机制在Spring是很重要的一部分内容，通过事件机制我们可以监听Spring容器中正在发生的一些事件，同样也可以自定义监听事件。Spring的事件为Bean和Bean之间的消息传递提供支持。当一个对象处理完某种任务后，通知另外的对象进行某些处理，常用的场景有进行某些操作后发送通知，消息、邮件等情况


```
private SpringApplicationRunListeners getRunListeners(String[] args) {
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
		return new SpringApplicationRunListeners(logger,
				getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
	}


private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```
这里的获取事件监听器同实例化的时候获取事件监听器一样


## 2.2 构造应用上下文环境
应用上下文环境包括什么呢？包括计算机的环境，Java环境，Spring的运行环境，Spring项目的配置（在SpringBoot中就是那个熟悉的application.properties/yml）等等

```
private ConfigurableEnvironment prepareEnvironment(
        SpringApplicationRunListeners listeners,
        ApplicationArguments applicationArguments) {
    // Create and configure the environment
    //创建并配置相应的环境
    ConfigurableEnvironment environment = getOrCreateEnvironment();
    //根据用户配置，配置 environment系统环境
    configureEnvironment(environment, applicationArguments.getSourceArgs());
    // 启动相应的监听器，其中一个重要的监听器 ConfigFileApplicationListener 就是加载项目配置文件的监听器。
    listeners.environmentPrepared(environment);
    bindToSpringApplication(environment);
    if (this.webApplicationType == WebApplicationType.NONE) {
        environment = new EnvironmentConverter(getClassLoader())
                .convertToStandardEnvironmentIfNecessary(environment);
    }
    ConfigurationPropertySources.attach(environment);
    return environment;
}
```
方法中主要完成的工作，首先是创建并配置相应的应用类型环境，然后根据用户的配置，配置系统环境，然后启动监听器，并加载系统配置文件

### 创建并配置相应的应用类型环境

```
private ConfigurableEnvironment getOrCreateEnvironment() {
		if (this.environment != null) {
			return this.environment;
		}
		switch (this.webApplicationType) {
		case SERVLET:
			return new StandardServletEnvironment();
		case REACTIVE:
			return new StandardReactiveWebEnvironment();
		default:
			return new StandardEnvironment();
		}
	}
```

### 配置系统环境

```
protected void configureEnvironment(ConfigurableEnvironment environment,
                                    String[] args) {
    // 将main 函数的args封装成 SimpleCommandLinePropertySource 加入环境中。
    configurePropertySources(environment, args);
    // 激活相应的配置文件
    configureProfiles(environment, args);
}
```

### 启动监听器，并加载系统配置文件

```
这个方法执行完毕会发现配置文件被加载
listeners.environmentPrepared(environment);
```
进入到方法一路跟下去就到了SimpleApplicationEventMulticaster类的multicastEvent()方法
![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191005222354.png)


查看getApplicationListeners(event, type)执行结果，发现一个重要的监听器ConfigFileApplicationListener，这个监听器默认的从 下面几个位置加载配置文件，并将其加入 上下文的 environment变量中，具体可以看注释

```
 * <ul>
 * <li>file:./config/:</li>
 * <li>file:./</li>
 * <li>classpath:config/</li>
 * <li>classpath:</li>
 * </ul>
```

## 2.3初始化上下文

```
protected ConfigurableApplicationContext createApplicationContext() {
		Class<?> contextClass = this.applicationContextClass;
		if (contextClass == null) {
			try {
				switch (this.webApplicationType) {
				case SERVLET:
					contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
					break;
				case REACTIVE:
					contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
					break;
				default:
					contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
				}
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Unable create a default ApplicationContext, " + "please specify an ApplicationContextClass",
						ex);
			}
		}
		return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
	}
```
根据应用类型，进行实例化


