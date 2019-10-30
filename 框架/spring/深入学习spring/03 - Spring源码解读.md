# 说明

spring的核心原理 就是 IOC、DI、AOP。



IOC就是控制反转，本来用户通过 new 来生成对象的方式 交给spring 去做，不用用户来手工生成对象了



spring 提供两种方式 xml 或者 注解 ，这里采用注解的方式，非xml。 因为 xml 需要的源码中包含很多解析xml的代码，注解的方式更加清爽



# 项目结构

```
.
├── pom.xml
├── spring_ioc.iml
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── jx
│   │   │           └── spring
│   │   │               └── ioc
│   │   │                   ├── bean
│   │   │                   │   └── Student.java
│   │   │                   └── test
│   │   │                       └── T1.java
```



## pom

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



## Javabean

```java
package com.jx.spring.ioc.bean;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

@Getter
@Setter
@ToString
@Component
public class Student {
    private Integer id;
    private String name;
}
```

## 测试代码

```java
package com.jx.spring.ioc.test;

import com.jx.spring.ioc.bean.Student;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class T1 {

    @Test
    public void t_01(){
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.jx.spring.ioc.bean");

        Student student = applicationContext.getBean(Student.class);
        student.setId(1);
        student.setName("a");
        System.out.println(student);
    }

}
```



### 测试结果

```java
十月 27, 2019 3:29:36 下午 org.springframework.context.support.AbstractApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@71f2a7d5: startup date [Sun Oct 27 15:29:36 CST 2019]; root of context hierarchy

Student(id=1, name=a)
```



# 入口类AnnotationConfigApplicationContext

## 

## 此类的定义

```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry
```

可以看到继承 GenericApplicationContext类 和 实现 AnnotationConfigRegistry接口

- GenericApplicationContext类 是通用Application上下文 ，实现了 BeanDefinitionRegistry 接口，BeanDefinitionRegistry接口主要提供 bean的 注册，bean的移除，获取bean信息等方法
- AnnotationConfigRegistry 接口有两个方法 register 和 scan



## 从构造方法开始跟踪

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191027161857.png)

构造方法有个片段，this 做了一些初始化工作， scan 做了扫描的操作 ， refresh 就是进行 application上下文的刷新了



### this

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191027162124.png)

实例化一个reader 和 scaner 对象，并给字段赋值







### scan

sacn的 主要功能就是 将 bean的 类文件定义存储起来了



![image-20191027162515821](/Users/no1/Library/Application Support/typora-user-images/image-20191027162515821.png)





![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191027162559.png)

```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
		Assert.notEmpty(basePackages, "At least one base package must be specified");
		Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
		for (String basePackage : basePackages) {
      
      // 这里会把扫描到的 bean 的元数据信息存起来，不是 bean 会被排除
			Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
			for (BeanDefinition candidate : candidates) {
        // 获取 bean的 作用域，默认是单例
				ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
				candidate.setScope(scopeMetadata.getScopeName());
        
        // 获取 bean 的名称
				String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
        
        // 此处为扫描的Bean，为ScannedGenericBeanDefinition，所以肯定为true
				// 注意：只是添加些默认的Bean定义信息，并不是执行后置处理器
				if (candidate instanceof AbstractBeanDefinition) {
					postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
				}
        
        // 此处为扫描的Bean，为ScannedGenericBeanDefinition，所以肯定为true
        // 也是完善比如Bean上的一些注解信息：比如@Lazy、@Primary 等
				if (candidate instanceof AnnotatedBeanDefinition) {
					AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
				}
        
        // 检查这个Bean, 没问题就会进入下面继续
				if (checkCandidate(beanName, candidate)) {
					BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
          
         //AnnotationConfigUtils类的applyScopedProxyMode方法根据注解Bean定义类中配置的作用域@Scope注解的值，为Bean定义应用相应的代理模式，主要是在Spring面向切面编程(AOP)中使用
					definitionHolder =
							AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
					beanDefinitions.add(definitionHolder);
          
          // 注意 注意 注意：这里已经吧Bean注册进去工厂了，注册的 bean 的定义，但是没有实例化bean
					registerBeanDefinition(definitionHolder, this.registry);
				}
			}
		}
		return beanDefinitions;
	}
```



### Refresh



这一段代码是最最核心了



```java
@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			//容器刷新前的准备，设置上下文状态，获取属性，验证必要的属性等
			prepareRefresh();

			// 获取新的beanFactory，销毁原有beanFactory、为每个bean生成BeanDefinition等  注意，此处是获取新的，销毁旧的，这就是刷新的意义
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			//配置标准的beanFactory，设置ClassLoader，设置SpEL表达式解析器等
			prepareBeanFactory(beanFactory);

			try {
				//模板方法，允许在子类中对beanFactory进行后置处理。
				postProcessBeanFactory(beanFactory);

				//实例化并调用所有注册的beanFactory后置处理器（实现接口BeanFactoryPostProcessor的bean）。
				//在beanFactory标准初始化之后执行  例如：PropertyPlaceholderConfigurer(处理占位符)
				invokeBeanFactoryPostProcessors(beanFactory);

				//实例化和注册beanFactory中扩展了BeanPostProcessor的bean。
				//例如：
				//AutowiredAnnotationBeanPostProcessor(处理被@Autowired注解修饰的bean并注入)
				//RequiredAnnotationBeanPostProcessor(处理被@Required注解修饰的方法)
				//CommonAnnotationBeanPostProcessor(处理@PreDestroy、@PostConstruct、@Resource等多个注解的作用)等。
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				//初始化国际化工具类MessageSource
				initMessageSource();

				// Initialize event multicaster for this context.
				//初始化事件广播器
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				//模板方法，在容器刷新的时候可以自定义逻辑（子类自己去实现逻辑），不同的Spring容器做不同的事情
				onRefresh();

				// Check for listener beans and register them.
				//注册监听器，并且广播early application events,也就是早期的事件
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				//非常重要。。。实例化所有剩余的（非懒加载）单例Bean。（也就是我们自己定义的那些Bean们）
				//比如invokeBeanFactoryPostProcessors方法中根据各种注解解析出来的类，在这个时候都会被初始化  扫描的 @Bean之类的
				//实例化的过程各种BeanPostProcessor开始起作用~~~~~~~~~~~~~~
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				//refresh做完之后需要做的其他事情
				//清除上下文资源缓存（如扫描中的ASM元数据）
				//初始化上下文的生命周期处理器，并刷新（找出Spring容器中实现了Lifecycle接口的bean并执行start()方法）。
				//发布ContextRefreshedEvent事件告知对应的ApplicationListener进行响应的操作
				finishRefresh();
			} catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				//如果刷新失败那么就会将已经创建好的单例Bean销毁掉
				destroyBeans();

				// Reset 'active' flag.
				//重置context的活动状态 告知是失败的
				cancelRefresh(ex);

				// Propagate exception to caller.
				//抛出异常
				throw ex;
			} finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				// 失败与否，都会重置Spring内核的缓存。因为可能不再需要metadata给单例Bean了。
				resetCommonCaches();
			}
		}
	}
```



#### 第一步：prepareRefresh()

```java
protected void prepareRefresh() {
		//记录容器启动时间，然后设立对应的标志位
		this.startupDate = System.currentTimeMillis();
		this.closed.set(false);
		this.active.set(true);

		// 打印info日志：开始刷新this此容器了
		if (logger.isInfoEnabled()) {
			logger.info("Refreshing " + this);
		}

		// 此处先忽略
		initPropertySources();

		// 这里其实就干了一件事，验证是否存在需要的属性
		getEnvironment().validateRequiredProperties();

		// 初始化容器，用于装载早期的一些事件
		this.earlyApplicationEvents = new LinkedHashSet<>();
	}
```



#### 第二步：obtainFreshBeanFactory()

实际上就是重新创建一个bean工厂，并销毁原工厂。主要工作是创建DefaultListableBeanFactory实例，解析配置文件，注册Bean的定义信息，这里用的 注解 走的 GenericApplicationContext， 如果是 XML 就会走AbstractApplicationContext（这个里面稍微复杂一些）

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191028092930.png)



#### 第三步：prepareBeanFactory(beanFactory)

这个方法是配置工厂的标准上下文特征

```java
	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// 设置beanFactory的classLoader为当前context的classLoader
		beanFactory.setBeanClassLoader(getClassLoader());
		// 设置EL表达式解析器（Bean初始化完成后填充属性时会用到）
		// spring3增加了表达式语言的支持，默认可以使用#{bean.xxx}的形式来调用相关属性值
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		// 设置属性注册解析器PropertyEditor 这个主要是对bean的属性等设置管理的一个工具
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
		
		// 将当前的ApplicationContext对象交给ApplicationContextAwareProcessor类来处理，从而在Aware接口实现类中的注入applicationContext等等
		// 添加了一个处理aware相关接口的beanPostProcessor扩展，主要是使用beanPostProcessor的postProcessBeforeInitialization()前置处理方法实现aware相关接口的功能
		// 类似的还有ResourceLoaderAware、ServletContextAware等等等等
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		// 下面是忽略的自动装配（也就是实现了这些接口的Bean，不要Autowired自动装配了）
		// 默认只有BeanFactoryAware被忽略,所以其它的需要自行设置
		// 因为ApplicationContextAwareProcessor把这5个接口的实现工作做了（具体你可参见源码） 所以这里就直接忽略掉
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// 设置几个"自动装配"规则======如下：
		// 如果是BeanFactory的类，就注册beanFactory
		//  如果是ResourceLoader、ApplicationEventPublisher、ApplicationContext等等就注入当前对象this(applicationContext对象)
		
		// 此处registerResolvableDependency()方法注意：它会把他们加入到DefaultListableBeanFactory的resolvableDependencies字段里面缓存这，供后面处理依赖注入的时候使用 DefaultListableBeanFactory#resolveDependency处理依赖关系
		// 这也是为什么我们可以通过依赖注入的方式，直接注入这几个对象比如ApplicationContext可以直接依赖注入
		// 但是需要注意的是：这些Bean，Spring的IOC容器里其实是没有的。beanFactory.getBeanDefinitionNames()和beanFactory.getSingletonNames()都是找不到他们的，所以特别需要理解这一点
		// 至于容器中没有，但是我们还是可以@Autowired直接注入的有哪些，请看下图：
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// 注册这个Bean的后置处理器：在Bean初始化后检查是否实现了ApplicationListener接口
		// 是则加入当前的applicationContext的applicationListeners列表 这样后面广播事件也就方便了
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));


		// 检查容器中是否包含名称为loadTimeWeaver的bean，实际上是增加Aspectj的支持
		// AspectJ采用编译期织入、类加载期织入两种方式进行切面的织入
		// 类加载期织入简称为LTW（Load Time Weaving）,通过特殊的类加载器来代理JVM默认的类加载器实现
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			// 添加BEAN后置处理器：LoadTimeWeaverAwareProcessor
        	// 在BEAN初始化之前检查BEAN是否实现了LoadTimeWeaverAware接口，
        	// 如果是，则进行加载时织入，即静态代理。
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// 注入一些其它信息的bean，比如environment、systemProperties、SystemEnvironment等
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}
```



#### 第四步：postProcessBeanFactory(beanFactory)

模版方法。因为beanFactory都准备好了，子类可以自己去实现自己的逻辑。
比如一些web的ApplicationContext，就实现了自己的逻辑，做一些自己的web相关的事情。如果是web环境下，因此会进来AbstractRefreshableWebApplicationContext#postProcessBeanFactory方法

#### 第五步：invokeBeanFactoryPostProcessors(beanFactory)

invokeBeanFactoryPostProcessors执行BeanFactory的后置处理器，**当然前提是你已经在容器中注册过此处理器了**



#### 第六步：registerBeanPostProcessors(beanFactory)

在讲解这个方法之前先看下什么是BeanPostProcessor，下面是BeanPostProcessor的代码

```java
// 在Bean实例化/依赖注入完毕以及自定义的初始化方法的 前后 调用。什么叫自定义初始化方法：比如init-method、比如@PostConstruct标、比如实现InitailztingBean接口的方法等等
// bean:这个Bean实例  beanName：bean名称
public interface BeanPostProcessor {

	// 在Bean实例化/依赖注入完毕以及自定义的初始化方法之前调用
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
	
	// 在Bean实例化/依赖注入完毕以及自定义的初始化方法之后调用
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}

```

**接口中两个方法不能返回null，如果返回null那么在后续初始化方法将报空指针异常或者通过getBean()方法获取不到bena实例对象 ，因为后置处理器从Spring IoC容器中取出bean实例对象没有再次放回IoC容器中**



我们从所有的@Bean定义中抽取出来了`BeanPostProcessor`然后都注册进去，等待后面的的顺序调用



#### 第七步：initMessageSource()

这部分逻辑比较简单：向容器里注册一个一个事件源的单例Bean：`MessageSource`



#### 第八步：initApplicationEventMulticaster()

初始化Spring的事件多播器：`ApplicationEventMulticaster`



#### 第九步：onRefresh()

类似于第四步的`postProcessBeanFactory`，它也是个模版方法，不同的环境下执行的就不一样



#### 第十步：registerListeners()

上面我们已经把事件源、多播器都注册好了，这里就是注册监听器了



####  第十一步：finishBeanFactoryInitialization(beanFactory)

- 简单看看执行前后的区别

在执行此方法之前，先看一下 beanFactory

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191028131937.png)

这里只有 10个 对象

执行完毕之后的截图如下：

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20191028132106.png)

可以看到 自定义的 bean ， Student 被实例化了。

- 方法概述

  这一步可谓和我们开发者打交道最多的，我们自定义的Bean绝大都是在这一步被初始化的，包括依赖注入等等

  ```java
  	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
  		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) && beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
  			beanFactory.setConversionService( beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
  		}
  
  		if (!beanFactory.hasEmbeddedValueResolver()) {
  			beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
  		}
  
  		// 注意此处已经调用了getBean方法，初始化LoadTimeWeaverAware Bean
  		// LoadTimeWeaverAware是类加载时织入的意思
  		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
  		for (String weaverAwareName : weaverAwareNames) {
  			getBean(weaverAwareName);
  		}
      
  		// 停止使用临时的类加载器
  		beanFactory.setTempClassLoader(null);
  
  		// 缓存（冻结）所有的bean definition数据，不期望以后会改变
  		beanFactory.freezeConfiguration();
  
  		// 这个就是最重要的方法：会把留下来的Bean们  不是lazy懒加载的bean都实例化掉
  		//  bean真正实例化的时刻到了
  		beanFactory.preInstantiateSingletons();
  	}
  ```

  接下来重点看看`DefaultListableBeanFactory#preInstantiateSingletons`：实例化所有剩余的单例Bean

  ```java
  	@Override
  	public void preInstantiateSingletons() throws BeansException {
  
  		// 此处目的，把所有的bean定义信息名称，赋值到一个新的集合中
  		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
  
  		for (String beanName : beanNames) {
  			// 这里先解释一下getMergedLocalBeanDefinition方法的含义，因为这个方法会常常看到。Bean定义公共的抽象类是AbstractBeanDefinition，普通的Bean在Spring加载Bean定义的时候，实例化出来的是GenericBeanDefinition，而Spring上下文包括实例化所有Bean用的AbstractBeanDefinition是RootBeanDefinition，这时候就使用getMergedLocalBeanDefinition方法做了一次转化，将非RootBeanDefinition转换为RootBeanDefinition以供后续操作
  			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
  	
  			// 不是抽象类&&是单例&&不是懒加载
  			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
  
  				// 这是Spring提供的对工程bean模式的支持：比如第三方框架的继承经常采用这种方式
  				// 如果是工厂Bean，那就会此工厂Bean放进去
  				if (isFactoryBean(beanName)) {
  					// 拿到工厂Bean本省，注意有前缀为：FACTORY_BEAN_PREFIX 
  					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
  					if (bean instanceof FactoryBean) {
  						final FactoryBean<?> factory = (FactoryBean<?>) bean;
  						boolean isEagerInit;
  						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
  							isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
  											((SmartFactoryBean<?>) factory)::isEagerInit,
  									getAccessControlContext());
  						} else {
  							isEagerInit = (factory instanceof SmartFactoryBean &&
  									((SmartFactoryBean<?>) factory).isEagerInit());
  						}
  
  						// true：表示渴望马上被初始化的，那就拿上执行初始化~
  						if (isEagerInit) {
  							getBean(beanName);
  						}
  					}
  				} else { 
  					// 这里，就是普通单例Bean正式初始化了~  核心逻辑在方法：doGetBean 
  					getBean(beanName);
  				}
  			}
  		}
  
  		// SmartInitializingSingleton：所有非lazy单例Bean实例化完成后的回调方法 Spring4.1才提供
  		//SmartInitializingSingleton的afterSingletonsInstantiated方法是在所有单例bean都已经被创建后执行的
  		//InitializingBean#afterPropertiesSet 是在仅仅自己被创建好了执行的
  		// 比如EventListenerMethodProcessor它在afterSingletonsInstantiated方法里就去处理所有的Bean的方法
  		// 看看哪些被标注了@EventListener注解，提取处理也作为一个Listener放到容器addApplicationListener里面去
  		for (String beanName : beanNames) {
  			Object singletonInstance = getSingleton(beanName);
  
  			if (singletonInstance instanceof SmartInitializingSingleton) {
  				final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
  				if (System.getSecurityManager() != null) {
  					AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
  						smartSingleton.afterSingletonsInstantiated();
  						return null;
  					}, getAccessControlContext());
  				}
  				else {
  					// 比如：ScheduledAnnotationBeanPostProcessor CacheAspectSupport  MBeanExporter等等
  					smartSingleton.afterSingletonsInstantiated();
  				}
  			}
  		}
  	}
  
  ```

  #### 第十二步：finishRefresh()

  上一步refresh做完之后需要做的其他事情

  ```java
  	protected void finishRefresh() {
  		// 这个是Spring5.0之后才有的方法
  		// 表示清除一些resourceCaches,如doc说的  清楚context级别的资源缓存，比如ASM的元数据
  		clearResourceCaches();
  
  		// 初始化所有的LifecycleProcessor
  		initLifecycleProcessor();
  
  		// 上面注册好的处理器，这里就拿出来，调用它的onRefresh方法了
  		getLifecycleProcessor().onRefresh();
  
  		// 发布容器刷新的事件：
  		publishEvent(new ContextRefreshedEvent(this));
  
  		// 和MBeanServer和MBean有关的。相当于把当前容器上下文，注册到MBeanServer里面去。
  		// 这样子，MBeanServer持久了容器的引用，就可以拿到容器的所有内容了，也就让Spring支持到了MBean的相关功能
  		LiveBeansView.registerApplicationContext(this);
  	}
  
  ```

  

