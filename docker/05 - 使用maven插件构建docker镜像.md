# 两种构建方式



构建镜像可以使用一下两种方式，第一种是将构建信息指定到 POM 中，第二种是使用已存在的 Dockerfile 构建。

- 第一种方式,支持将 `FROM`, `ENTRYPOINT`, `CMD`, `MAINTAINER` 以及 `ADD` 信息配置在 POM 中，不需要使用 `Dockerfile` 配置
- 第二种方式,如果使用 `VOLUME` 或其他 `Dockerfile` 中的命令的时候，需要创建一个 `Dockerfile`，并在 POM 中配置 `dockerDirectory` 来指定路径即可



## 第一种：不需要dockerfile

完整pom文件如下

```
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
    <groupId>com.jx.learn.docker</groupId>
    <artifactId>spring_boot_hello</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring_boot_hello</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            
            <!-- 这里就是加入的 -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <configuration>
                    <!-- 指定镜像名称 -->
                    <imageName>${project.artifactId}:${project.version}</imageName>
                    <!-- 指定基础镜像，类似Dockerfile 中的 FROM -->
                    <baseImage>java</baseImage>
                    <cmd>["java", "-version"]</cmd>
                    <!-- 类似Dockerfile中的 entrypoint -->
                    <entryPoint>["java", "-jar", "${project.build.finalName}.jar"]</entryPoint>
                    <!-- 这里是复制 jar 包到 docker 容器指定目录配置 -->
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <!--要复制的根目录-->
                            <directory>${project.build.directory}</directory>
                            <!--指定要复制的文件-->
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

执行maven命令，注意要在pom.xml文件同目录，否则报错

```
mvn clean package docker:build
```



## 第二种：使用dockerfile构建

pom文件

```xml
<plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <configuration>
                    <imageName>${project.artifactId}:${project.version}</imageName>
                    <!-- 指定 Dockerfile 路径-->
                    <dockerDirectory>${project.basedir}</dockerDirectory>
                    <!-- 这里是复制 jar 包到 docker 容器指定目录配置，也可以写到 Docokerfile 中 -->
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
```



dockerfile文件（在项目根目录，与pom.xml同目录）

```
# 基于jdk8镜像
FROM java:8

# 将本地文件夹挂载到当前容器
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp

# 将jar包添加到容器中并更名
ADD target/spring_boot_hello-0.0.1-SNAPSHOT.jar app.jar

# 运行jar包
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

执行maven命令，注意要在pom.xml文件同目录，否则报错

```
mvn clean package docker:build
```



# 自动构建docker镜像

绑定 Docker 命令到 Maven 各个阶段，可以把 Docker 分为 build、tag、push，然后分别绑定 Maven 的 package、deploy 等阶段，此时，我们只需要执行 `mvn deploy` 就可以完成整个 build、tag、push 操作了，当我们执行 `mvn build` 就只完成 build、tag 操作。此外当想跳过某些步骤或者只执行某个步骤时，不需要修改 POM 文件，只需要指定跳过 docker 某个步骤即可。比如当工程已经配置好了自动化模板了，但是这次我们只需要打镜像到本地自测，不想执行 push 阶段，那么此时执行要指定参数 `-DskipDockerPush` 就可跳过 push 操作了





## 测试

上面都是 执行 mvn clean package docker:build 命令来进行的构建，现在改成 执行完毕 clean package 就进行构建



完整pom 如下：实际上只需要 加入    

​			<executions>
​                    <execution>
​                        <id>build-image</id>
​                        <phase>package</phase>
​                        <goals>
​                            <goal>build</goal>
​                        </goals>
​                    </execution>
​                </executions>

```
<plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <executions>
                    <execution>
                        <id>build-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <imageName>${project.artifactId}:${project.version}</imageName>
                    <!-- 指定 Dockerfile 路径-->
                    <dockerDirectory>${project.basedir}</dockerDirectory>
                    <!-- 这里是复制 jar 包到 docker 容器指定目录配置，也可以写到 Docokerfile 中 -->
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
```



## 扩展

```
<executions>
                <execution>
                    <id>build-image</id>
                    <phase>package</phase>
                    <goals>
                        <goal>build</goal>
                    </goals>
                </execution>
                <execution>
                    <id>tag-image</id>
                    <phase>package</phase>
                    <goals>
                        <goal>tag</goal>
                    </goals>
                </execution>
                <execution>
                    <id>push-image</id>
                    <phase>deploy</phase>
                    <goals>
                        <goal>push</goal>
                    </goals>
                </execution>
            </executions>
```

当我们执行 `mvn package` 时，执行 build、tag 操作，当执行 `mvn deploy` 时，执行 build、tag、push 操作。如果想跳过 docker 某个过程时，只需要：

- -DskipDockerBuild 跳过 build 镜像
- -DskipDockerTag 跳过 tag 镜像
- -DskipDockerPush 跳过 push 镜像
- -DskipDocker 跳过整个阶段

例：我们想执行 package 时，跳过 tag 过程，那么就需要 `mvn package -DskipDockerTag`





# 自动推送镜像到仓库（略）