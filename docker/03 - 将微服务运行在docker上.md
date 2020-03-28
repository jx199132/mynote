# 使用dockerfile构建docker镜像_以Nginx入门

## 准备工作

新建一个文件夹

```shell
mkdir -p /home/mydockerfile/nginx_docker_file 
cd /home/mydockerfile/nginx_docker_file/
```



## 新建一个 Dockerfile

```shell
vim Dockerfile

# 下面是 dockerfile中的内容
FROM nginx
RUN echo '<h1> my first dockerfile </h1>' > /usr/share/nginx/html/index.html

```

## 构建Dockerfile

```shell
[root@localhost nginx_docker_file]# docker build -t nginx:my .
Sending build context to Docker daemon 2.048 kB
Step 1/2 : FROM nginx
latest: Pulling from library/nginx
8d691f585fa8: Pull complete 
5b07f4e08ad0: Pull complete 
abc291867bca: Pull complete 
Digest: sha256:922c815aa4df050d4df476e92daed4231f466acc8ee90e0e774951b0fd7195a4
Status: Downloaded newer image for nginx:latest
 ---> 540a289bab6c
Step 2/2 : RUN echo '<h1> my first dockerfile </h1>' > /usr/share/nginx/html/index.html
 ---> Running in 0a8ff54f061c
 ---> c2f69468d36e
Removing intermediate container 0a8ff54f061c
Successfully built c2f69468d36e
```

注意后面有一个 点

- -t nginx:my    前面的 nginx 表示 镜像的名称，  后面的 my 表示 版本
- . 最后的点 表示 当前目录



查看构建完毕的镜像

```shell
[root@localhost nginx_docker_file]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               my                  c2f69468d36e        34 seconds ago      126 MB
```



## 使用构建的镜像，启动容器

```shell
docker run -d -p 92:80 nginx:my 
```



通过浏览器测试 http://172.16.72.150:92/

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191101143516.png)





# 使用dockerfile构建docker镜像_使用spring boot项目

## 新建一个spring boot项目

```
.
├── pom.xml
├── spring_boot_hello.iml
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── jx
│   │   │           └── learn
│   │   │               └── docker
│   │   │                   └── spring_boot_hello
│   │   │                       ├── SpringBootHelloApplication.java
│   │   │                       └── controller
│   │   │                           └── HelloController.java
│   │   └── resources
│   │       ├── application.properties
│   │       ├── static
│   │       └── templates
```



### pom文件

新建的spring boot项目，默认勾选了 web

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### application.properties

就加上了端口  8888

### SpringBootHelloApplication

默认值不变

### HelloController

```java
@RestController
@RequestMapping("hello")
public class HelloController {

    @RequestMapping("index")
    public String index(String name){
        System.out.println("name is " + name);
        return "hello : " + name;
    }

}
```



## 通过maven将项目打包

```
mvn clean package
```

这是本项目中打包完毕的 jar 路径

/Users/no1/work/idea/docker/spring_boot_hello/target/spring_boot_hello-0.0.1-SNAPSHOT.jar



## 编写dockerfile

在项目的根目录下  新建 Dockerfile 文件 

```
├── Dockerfile
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
└── target
    ├── spring_boot_hello-0.0.1-SNAPSHOT.jar

```

文件内容如下

```dockerfile
# 基于jdk8镜像
FROM java:8

# 将本地文件夹挂载到当前容器
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp

# 将jar包添加到容器中并更名 , 注意这里的 app.jar 和 下面是对应的
# <src> 必须是想对于源文件夹的一个文件或目录，也可以是一个远程的url。<dest> 是目标容器中的绝对路径
ADD target/spring_boot_hello-0.0.1-SNAPSHOT.jar app.jar

# 运行jar包
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```



## 使用dockerfile构建镜像

```shell
$ cd /Users/no1/work/idea/docker/spring_boot_hello/src/main/resources

# 格式 docker build -t 仓库/镜像名称:版本  dockerfile相对路径 
$ docker build -t learn/spring_boot_hello:0.0.1 .

$ docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
learn/spring_boot_hello   0.0.1               e040b49d4d98        8 seconds ago       677MB
```



## 以打包的镜像启动容器

```shell
$ docker run -d -p 7777:8888 --name spring_boot_hello learn/spring_boot_hello:0.0.1
```

不带 -d  即可 输出控制台内容

```
$ docker run -p 7777:8888 --name spring_boot_hello learn/spring_boot_hello:0.0.1

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.9.RELEASE)

2019-11-01 13:56:21.637  INFO 1 --- [           main] c.j.l.d.s.SpringBootHelloApplication     : Starting SpringBootHelloApplication v0.0.1-SNAPSHOT on 83895a91c3ca with PID 1 (/app.jar started by root in /)
2019-11-01 13:56:21.639  INFO 1 --- [           main] c.j.l.d.s.SpringBootHelloApplication     : No active profile set, falling back to default profiles: default
2019-11-01 13:56:22.754  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8888 (http)
2019-11-01 13:56:22.791  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]

```



![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191101215655.png)



