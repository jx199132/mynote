# 概述

### 文件描述符

与文件输入、输出相关联的整数。它用来跟踪已经打开的文件。最常见的文件描述符是stdin、stdout、stderr。

我们甚至可以将某个文件描述的内容重定向到另一个文件描述符中。文件描述符0，1，2是系统预留的：

0----stdin（标准输入）　　1----stdout（标准输出）　　2----stderr(标准错误)



# 输出重定向 >,>>

表示把将输出重定向到屏幕或者设备或者文件

### 示例

将输出的文本重定向到一个文件中
```
 echo "ho" > b.txt
```
将文本追加到目标文件

```
echo "ho" >> b.txt
```

### 标准错误文件描述符的重定向
当命令输出错误信息时，stderr信息就会被打印出来

```
[root@localhost shell]# ls +
ls: 无法访问 +: 没有那个文件或目录
[root@localhost shell]# echo $?
2
[root@localhost shell]# ls + > out.txt
ls: 无法访问 +: 没有那个文件或目录
[root@localhost shell]# ls + 2> out.txt
[root@localhost shell]# cat out.txt
ls: 无法访问 +: 没有那个文件或目录
```
说明：

第一次的ls + 中的+是非法参数，因此命令将执行失败

通过echo $? 输出上个命令执行后状态

第三句的ls + >out.txt将执行失败的信息输出到了屏幕，并不是文件中

最后一句加上标准错误2 才输出到了文件中，然后可以查看到文件的内容




# 输入重定向 <,<<

```
[root@web01 datas]# cat >> 123.txt << EOF
> 123
> fdsafa
> EOF
[root@web01 datas]# cat 123.txt 
123
fdsafa

```
将输入的内容输出到123.txt，遇到EOF终止输入