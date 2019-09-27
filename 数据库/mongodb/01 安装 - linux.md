### 下载

wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.4.tgz



### 解压

复制到use/local文件夹下

```sh
cp mongodb-linux-x86_64-3.4.4.tgz /usr/local/
```



解压

```shell
tar -zxvf mongodb-linux-x86_64-3.4.4.tgz 
```





**配置**

- 解压完毕之后目录 存在下面的文件与文件夹 

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20190927160605.png)

- 创建data文件夹与log文件 

/usr/local/mongodb-linux-x86_64-3.4.4/data 

/usr/local/mongodb-linux-x86_64-3.4.4/data/db 

/usr/local/mongodb-linux-x86_64-3.4.4/mongodb.log



- 创建一个配置文件mongodb.conf

```
cd /usr/local/mongodb-linux-x86_64-3.4.4/
vi mongodb.conf
```

配置文件内容如下图： 

![](https://raw.githubusercontent.com/jx199132/pic/master/pic/20190927161004.png)



```
dbpath指定存放地址 

logpath指定日志文件 

port指定端口 

rest 这里没写， 标志是否开启简单rest接口，增强服务器默认的web控制台，如果开启 那么在浏览器输入http://localhost:27018 可以看到控制台很多信息 

fork 是否以守护进程的方式启动
```



### **启动**

切换目录到MongoDB的 bin下面

```shell
# -f 表示引入配置文件的方式启动
./mongod -f ../mongodb.conf
```







