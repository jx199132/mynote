 **下载**

https://fastdl.mongodb.org/win32/mongodb-win32-x86_64-2008plus-ssl-3.4.18-signed.msi



 **安装**

- 点击安装包 选择目录 E:\mongodb3.4目录下
- 在E:\mongodb3.4目录下 新建一个文件夹data , 然后在data中新建两个文件夹（db,log）
- 启动CMD，以管理员模式
- cd切换到 E:\mongodb3.4\bin目录下
- E:\mongodb3.4\bin>mongod.exe --dbpath "e:\mongodb3.4\data\db" --logpath "e:\mongodb3.4\data\log\mongodb.log" --bind_ip 192.168.18.225 --port 27017 --serviceName "MongoDB" --install



--bind_ip”：PC机的真实IP地址

“--logpath”：创建的数据库日志文件路径

“--dbpath”：创建的数据存放路径

“--port”：数据库占用的端口，默认的是27017；

“--serviceName”：windows服务名称



**启动**

直接在window自带的服务中启动即可