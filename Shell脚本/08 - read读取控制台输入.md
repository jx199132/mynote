
# 基本语法


```
read(选项)(参数)
	选项：
-p：指定读取值时的提示符；
-t：指定读取值时等待的时间（秒）。
参数
	变量：指定读取值的变量名
```


# 案例

提示7秒内，读取控制台输入的名称，如果超过七秒不输入内容，那么就不等待了，执行后面的代码


```
[root@web01 datas]# touch read.sh
[root@web01 datas]# vim read.sh 

···
#!/bin/bash

read -t 7 -p "请输入你的名字" NAME

# 如果七秒内没有输入，那么只会打印出来 你好
echo "你好！ $NAME"

···

[root@web01 datas]# sh read.sh 
请输入你的名字九霄
你好！ 九霄


```
