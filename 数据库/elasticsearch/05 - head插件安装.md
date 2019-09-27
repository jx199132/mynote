# 注意：

不需要再elasticsearch的机器上装，任意一台机器安装都可以，然后连接elasticsearch的机器即可。

# 安装Node环境

下载
```
wget http://nodejs.org/dist/v8.1.4/node-v8.1.4-linux-x64.tar.gz
```

安装 node 需要的基础环境

```
yum -y install gcc make gcc-c++ openssl-devel wget
```

解压 node

```
tar -xf node-v8.1.4-linux-x64.tar.gz

mv node-v8.1.4-linux-x64 /usr/local/node

cd /usr/local/node/


# 可以看到目录结构
[root@localhost node]# ls
bin  CHANGELOG.md  include  lib  LICENSE  README.md  share

```
设置nodejs环境变量


```
vim /etc/profile
    export NODE_HOME=/usr/local/node
    export PATH=$NODE_HOME/bin:$PATH
:wq
source /etc/profile
```

验证node安装

```
node -v
```

# 安装elasticsearch-head
## 下载

```
cd /home
wget https://github.com/mobz/elasticsearch-head/archive/master.zip
```

## 解压

```
unzip master.zip
cd elasticsearch-head-master/

npm install -g grunt --registry=https://registry.npm.taobao.org
```

## 下载phantomjs-2.1.1-linux-x86_64.tar.bz2

由于linux下载比较慢，通过window机器下载文件

https://github.com/Medium/phantomjs/releases/download/v2.1.1/phantomjs-2.1.1-linux-x86_64.tar.bz2

将该文件放到linux机器的/tmp/phantomjs/phantomjs-2.1.1-linux-x86_64.tar.bz2

## 为head插件安装node依赖
```
npm install
```
如果出现错误“npm ERR! phantomjs-prebuilt@2.1.16 install: `node install.js`”


```
npm install phantomjs-prebuilt@2.1.16 --ignore-scripts

npm install
```



## 修改配置文件
1. elasticsearch-head-master/Gruntfile.js

增加hostname属性，设置为*
```
connect: {
    server: {
        options: {
            port: 9100,
            hostname: '*',
            base: '.',
            keepalive: true
        }
    }
}
```

2. elasticsearch-head-master/_site/app.js

```
修改前：

this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9200";

修改后：

this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://192.168.204.129:9200";
```


# 修改elasticsearch配置文件

修改elasticsearch.yml文件加入以下内容：
```
# 是否支持跨域
http.cors.enabled: true

# *表示支持所有域名
http.cors.allow-origin: "*"
```


# 启动head

先启动elasticsearch，在启动head插件
```
grunt server
```
