# 拆解spring boot项目的依赖

## 新建一个springboot项目

不做任何添加、不配置配置文件、仅仅在pom中引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.jx.learn.springboot</groupId>
    <artifactId>boot_01</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>boot_01</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

到这里一个 spring boot 的 web项目就建立好了



### 几个关键点

- 继承了 spring-boot-parent
- 引入依赖spring-boot-starter
- 引入依赖spring-boot-starter-web



### 测试打包

#### 打 jar 包

```
<packaging>jar</packaging>
```

通过maven打包

```
[INFO] Building jar: /Users/no1/work/idea/springboot_learn/boot_01/target/boot_01-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.1.9.RELEASE:repackage (repackage) @ boot_01 ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

启动

```
java -jar /Users/no1/work/idea/springboot_learn/boot_01/target/boot_01-0.0.1-SNAPSHOT.jar
```





#### 打war包

```
<packaging>war</packaging>
```

通过maven打包

```
[INFO] Building war: /Users/no1/work/idea/springboot_learn/boot_01/target/boot_01-0.0.1-SNAPSHOT.war
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.1.9.RELEASE:repackage (repackage) @ boot_01 ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

启动

```
java -jar /Users/no1/work/idea/springboot_learn/boot_01/target/boot_01-0.0.1-SNAPSHOT.war
```



到这里一切都是正常的



## 改造依赖

上面新建的项目 继承了 spring-boot-starter-parent，但是往往企业内部会有一些自己内部的jar需要依赖，需要使用企业自己的parent

###   拿掉spring-boot-starter-parent

注释掉这一段

```
		<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```



### 加入依赖 spring-boot-dependencies

```
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.9.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```



### 加入打包依赖

```xml
			<plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
              	
                <!-- 版本号不加，会报错 -->
              	<!-- 版本号来自于  spring-boot-dependencies pom 文件中 -->
                <version>2.1.9.RELEASE</version>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

        </plugins>
```



### 测试打包

#### 打jar包并启动

```
java -jar boot_01-0.0.1-SNAPSHOT.jar 
boot_01-0.0.1-SNAPSHOT.jar中没有主清单属性
```

发现 打包通过，但是无法启动

仔细观察打包输出并没有执行插件 spring-boot-maven-plugin

```
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------< com.jx.learn.springboot:boot_01 >-------------------
[INFO] Building boot_01 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ boot_01 ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ boot_01 ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 2 source files to /Users/no1/work/idea/springboot_learn/boot_01/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ boot_01 ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/no1/work/idea/springboot_learn/boot_01/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ boot_01 ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /Users/no1/work/idea/springboot_learn/boot_01/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ boot_01 ---
[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ boot_01 ---
[INFO] Building jar: /Users/no1/work/idea/springboot_learn/boot_01/target/boot_01-0.0.1-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```



改造打包插件

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  <version>2.1.9.RELEASE</version>
  <executions>
    <execution>
      <goals>
      	<goal>repackage</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

重新打包，查看输出日志

```
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ boot_01 ---
[INFO] Building jar: /Users/no1/work/idea/springboot_learn/boot_01/target/boot_01-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.1.9.RELEASE:repackage (default) @ boot_01 ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

可以看到非常明显的  [INFO] --- spring-boot-maven-plugin:2.1.9.RELEASE:repackage，说明该插件执行了



再次启动，一切正常

```
java -jar boot_01-0.0.1-SNAPSHOT.jar 
```



#### 打war包并启动过

在上面打war包的基础上，什么都不做，仅仅修改 <package>jar<package> ，改成 <package>war</package>



打包报错：**[ERROR] Failed to execute goal org.apache.maven.plugins:maven-war-plugin:2.2:war**



继续在 pom中加入  plugin

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-war-plugin</artifactId>
  <version>3.2.3</version>
</plugin>
```



重新打包，测试，一切ok。



### 总结

新建的spring_boot项目 默认 继承了 spring-boot-starter-parent依赖，往往企业内部会有一些自己内部的jar需要依赖，需要使用企业自己的parent，那么改造 pom



- 加入spring-boot-dependencies
- 给spring-boot-maven-plugin 加上版本号（来自 spring-boot-dependencies 中)
- 给spring-boot-maven-plugin  加上 <goal>repackage</goal>
- 加入maven-war-plugin ，版本号 来自 spring-boot-dependencies 中



完整pom如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.jx.learn.springboot</groupId>
    <artifactId>boot_01</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>boot_01</name>
    <packaging>war</packaging>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

		<!-- 加入 spring-boot-dependencies 开始-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.9.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
		<!-- 加入 spring-boot-dependencies 结束-->
		
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
          	<!-- 加入 maven-war-plugin 开始-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.2.3</version>
            </plugin>
						<!-- 加入 maven-war-plugin 结束-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
              	<!-- 加入 版本号 -->
                <version>2.1.9.RELEASE</version>
                <executions>
                    <execution>
                        <goals>
                          	<!-- 加入 repackage -->
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>

```

