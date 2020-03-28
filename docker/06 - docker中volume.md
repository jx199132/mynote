## 说明

volume也是绕过container的文件系统，直接将数据写到host机器上，只是volume是被docker管理的，docker下所有的volume都在host机器上的指定目录下/var/lib/docker/volumes



### 直接通过docker命令挂载



#### 指定挂载目录

将my-volume挂载到container中的/mydata目录

```
docker run -it -v my-volume:/mydata alpine sh
```

然后可以查看到给my-volume的volume

```
docker volume inspect my-volume
[
    {
        "CreatedAt": "2018-03-28T14:52:49Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/my-volume/_data",
        "Name": "my-volume",
        "Options": {},
        "Scope": "local"
    }
]
```



可以看到，volume在host机器的目录为`/var/lib/docker/volumes/my-volume/_data`。此时，如果my-volume不存在，那么docker会自动创建my-volume，然后再挂载



#### 不指定挂载目录

也可以不指定host上的volume：

```
docker run -it -v /mydata alpine sh
```

此时docker将自动创建一个匿名的volume，并将其挂载到container中的/mydata目录。匿名volume在host机器上的目录路径类似于：`/var/lib/docker/volumes/300c2264cd0acfe862507eedf156eb61c197720f69e7e9a053c87c2182b2e7d8/_data`



#### 注意的点

需要注意的是，与bind mount不同的是，如果volume是空的而container中的目录有内容，那么docker会将container目录中的内容拷贝到volume中，但是如果volume中已经有内容，则会将container中的目录覆盖






### Dockerfile中的VOLUME

在Dockerfile中，我们也可以使用VOLUME指令来申明contaienr中的某个目录需要映射到某个volume

```bash
#Dockerfile
VOLUME /foo
```

在docker运行时，docker会创建一个匿名的volume，并将此volume绑定到container的/foo目录中，如果container的/foo目录下已经有内容，则会将内容拷贝的volume中
