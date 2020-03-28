# 环境说明

- mac机器

- centos7机器
- 阿里云docker镜像仓库



## 目标



mac机器 发布镜像——》推送到 阿里云镜像仓库——》centos7机器 从 阿里云镜像仓库下载



通过上面流程，达到  多台机器共享 阿里云 镜像仓库



# 阿里云镜像仓库配置

1	首先登录阿里云账号

2	找到**容器镜像服务**，	找到镜像仓库，设置密码

3	创建一个镜像仓库（填写命名空间：jx_ck，填写仓库名称：my_docker_ck，填写摘要：docker仓库）

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191104103447.png)

4	下一步，选择私有仓库，点击完成创建



## 登录阿里云docker 

```shell
$ docker login --username=jiuxiao199132@gmail.com registry.cn-hangzhou.aliyuncs.com
Password: （提示输入密码，就是上面设置的密码）
Login Succeeded
```

## 从mac机器将镜像推送到Registry

本地现有 两个 镜像 如下：

```shell
$ docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
learn/spring_boot_hello   0.0.1               4a9a6b2f48cc        3 seconds ago       677MB
java                      8                   d23bdf5b1b1b        2 years ago         643MB
```



```

# 语法 docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck:[镜像版本号]

# 使用"docker tag"命令重命名镜像，并将它通过专有网络地址推送至Registry
$ docker tag 4a9a6b2f48cc registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck:learn_spring_boot_hello_0.0.1
$ docker tag d23bdf5b1b1b registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck:java_8

$ docker push registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck:learn_spring_boot_hello_0.0.1
$ docker push registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck:java_8
```



上传成功之后的 截图

![](https://raw.githubusercontent.com/jx199132/pic/master/pic01/20191104135734.png)



# 从centos机器拉取镜像

## 登录阿里云docker

```shell
# docker login --username=jiuxiao199132@gmail.com registry.cn-hangzhou.aliyuncs.com
Password: 
Login Succeeded
```



## 从阿里云镜像仓库拉取镜像

```sh
# docker pull registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck:learn_spring_boot_hello_0.0.1

# docker pull registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck:java_8
```





# 测试



## 先查看镜像

```shell
[root@localhost ~]# docker images
REPOSITORY                                             TAG                             IMAGE ID            CREATED             SIZE
registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck   learn_spring_boot_hello_0.0.1   4a9a6b2f48cc        About an hour ago   677 MB
spring_boot_hello                                      latest                          c47e48d1befe        2 days ago          660 MB
my_nginx                                               my                              c2f69468d36e        2 days ago          126 MB
nginx                                                  my                              c2f69468d36e        2 days ago          126 MB
nginx                                                  latest                          540a289bab6c        12 days ago         126 MB
java                                                   8                               d23bdf5b1b1b        2 years ago         643 MB
registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck   java_8                          d23bdf5b1b1b        2 years ago         643 MB

```

发现有 **registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck**  仓库下有两个镜像



## 运行spring_boot_hello镜像

```shell
# registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck 表示仓库名字
# learn_spring_boot_hello_0.0.1 表示版本号
# 上面两者联合确定镜像
docker run -p 7777:8888 --name test_pull_spring_boot_hello registry.cn-hangzhou.aliyuncs.com/jx_ck/my_docker_ck:learn_spring_boot_hello_0.0.1
```



## 浏览器测试

http://172.16.72.150:7777/hello/index?name=123

结果ok