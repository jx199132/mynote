# springboot项目如何启动的



## 启动方式

- 在ide中可以执行运行代码启动


```java
@SpringBootApplication
public class RunByJarApplication {

    public static void main(String[] args) {
        SpringApplication.run(RunByJarApplication.class, args);
    }

}
```



- 通过maven打包之后的项目，也可以通过命令启动


```shell
java -jar /Users/no1/work/idea/springboot/run_by_jar/target/run_by_jar-0.0.1-SNAPSHOT.jar
```

这里就比较奇怪了，通过maven打包之后 仅仅输入了命令 mvn package。并没有指定main方法，那么是怎么启动的呢？



将jar包进行解压，查看解压出来的 META-INF/MANIFEST.MF

```
cd /Users/no1/work/idea/springboot/run_by_jar/target/
unzip run_by_jar-0.0.1-SNAPSHOT.jar 

文件中内容如下
Manifest-Version: 1.0
Implementation-Title: run_by_jar
Implementation-Version: 0.0.1-SNAPSHOT

# 这里还有一个 开始类
Start-Class: com.jx.run_by_jar.RunByJarApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Build-Jdk-Spec: 1.8
Spring-Boot-Version: 2.1.9.RELEASE
Created-By: Maven Archiver 3.4.0

# 可以看到这里有 指定 main 方法的入口
Main-Class: org.springframework.boot.loader.JarLauncher
```



## maven打包的过程

从ide查看项目发现并没有 org.springframework.boot.loader.JarLauncher ，从maven打出来 run_by_jar-0.0.1-SNAPSHOT.jar 中发现会有一个 org.springframework.boot.loader.JarLauncher。  所以这里的应该是maven的打包插件干的，如果想查看maven打包是做了些什么，可以下载插件源码进行查看

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>2.1.8.RELEASE</version>
</dependency>
```





## spring-boot-loader过程

通过java -jar archive.jar 运行时候Launcher会去加载JarLauncher类并执行其中的main函数，JarLauncher主要关心构造一个合适的URLClassLoader加载器用来调用我们应用程序的main方法

详情可以通过maven下载依赖查看源码

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-loader</artifactId>
    <version>2.1.9.RELEASE</version>
    <scope>provided</scope>
</dependency>
```



下面是spring-boot-loader的片段代码

```java
public void run() throws Exception {
		Class<?> mainClass = Thread.currentThread().getContextClassLoader().loadClass(this.mainClassName);
		Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
		mainMethod.invoke(null, new Object[] { this.args });
	}
```

