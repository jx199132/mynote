# cat
cat 命令用于连接文件并打印到标准输出设备上

##### 语法
```
cat [-AbeEnstTuv] [--help] [--version] fileName
```

##### 参数说明
-n 或 --number：由 1 开始对所有输出的行数编号。

-b 或 --number-nonblank：和 -n 相似，只不过对于空白行不编号。

-s 或 --squeeze-blank：当遇到有连续两行以上的空白行，就代换为一行的空白行。

-v 或 --show-nonprinting：使用 ^ 和 M- 符号，除了 LFD 和 TAB 之外。

-E 或 --show-ends : 在每行结束处显示 $。

-T 或 --show-tabs: 将 TAB 字符显示为 ^I。

-A, --show-all：等价于 -vET。

-e：等价于"-vE"选项；

-t：等价于"-vT"选项；

##### 案例
把 textfile1 的文档内容加上行号后输入 textfile2 这个文档里

```
cat -n textfile1 > textfile2
```






# cut
cut的工作就是“剪”，具体的说就是在文件中负责剪切数据用的。cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段输出

##### 语法


```
cut [选项参数]  filename
```

##### 参数说明


选项参数| 功能
---|---
-f | 列号，提取第几列
-d | 分隔符，按照指定分隔符分割列

###### -f 说明

```
-f 1 #提取第一列
-f 2,3 #提取第二列和第三列
-f 3- #提取第三列和之后所有列
```

##### 案列

数据准备

```
[root@web01 datas]# touch cut.txt
[root@web01 datas]# vim cut.txt 


a 1 你
b 2 好
c 3 啊
d 4 !


```

切割cut.txt第一列

```
[root@web01 datas]# cut cut.txt -d " " -f 1
a
b
c
d

```

切割cut.txt第二列和第三列

```
[root@web01 datas]# cut cut.txt -d " " -f 2,3
1 你
2 好
3 啊
4 !
```

切割cut.txt第二列和之后的所有列

```
[root@web01 datas]# cut cut.txt -d " " -f 2-
1 你
2 好
3 啊
4 !
```

切割出来cut.txt中的 “好” : ==这里用到了 cat , grep , cut三个命令组合==

```
[root@web01 datas]# cat cut.txt
a 1 你
b 2 好
c 3 啊
d 4 !
[root@web01 datas]# cat cut.txt | grep "你"
a 1 你
[root@web01 datas]# cat cut.txt | grep "你" | cut -d " " -f 3
你

```

选取系统PATH变量值，第2个“：”开始后的所有路径

```
[root@web01 datas]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/jdk1.8.0_191/bin:/root/bin
[root@web01 datas]# echo $PATH | cut -d ":" -f 3-
/usr/sbin:/usr/bin:/usr/local/jdk1.8.0_191/bin:/root/bin
```


# sed

sed是一种流编辑器，==它一次处理一行内容==。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。==文件内容并没有改变==，除非你使用重定向存储输出

##### 基本用法

选项参数 | 功能
---|---
-e | 直接在指令列模式上进行sed的动作编辑


##### 命令功能描述(部分)

命令 | 功能描述
---|---
a | 新增，a的后面可以接字串，在下一行出现
d | 删除
s | 查找并替换

##### 案例

数据准备

```
[root@web01 datas]# touch s.txt
[root@web01 datas]# vim s.txt 

你好
abcd
aaaa
测试
sed
```

将“shell”这个单词插入到第二行下

```
[root@web01 datas]# sed "2a shell" s.txt 
你好
abcd
shell
aaaa
测试
sed
[root@web01 datas]# cat s.txt 
你好
abcd
aaaa
测试
sed
```

删除第二行数据

```
[root@web01 datas]# sed "2d" s.txt 
你好
aaaa
测试
sed
```

删除包含“a”的行

```
[root@web01 datas]# sed "/a/d" s.txt 
你好
测试
sed
```

将"a"替换成"b"

```
[root@web01 datas]# sed "s/a/b/g" s.txt   # g代表全部替换，否则只替换每一行第一个
你好
bbcd
bbbb
测试
sed

```


```
[root@web01 datas]# sed "s/a/b/" s.txt 
你好
bbcd
baaa
测试
sed
```

将第四行删除，并且将 a 替换为 x

```
[root@web01 datas]# sed -e "4d" -e "s/a/x/g" s.txt 
你好
xbcd
xxxx
sed

```

# awk

一个强大的文本分析工具，==把文件逐行的读入==，以空格为默认分隔符将每行切片，切开的部分再进行分析处理

##### 基本用法


```
awk [选项参数] ‘pattern1{action1}  pattern2{action2}...’ filename
```
pattern：表示AWK在数据中查找的内容，就是匹配模式

action：在找到匹配内容时所执行的一系列命令

##### 选项参数说明

选项参数 | 功能
---|---
-F | 指定输入文件折分隔符
-v | 赋值一个用户定义变量

##### 案例

###### 数据准备

```
[root@web01 datas]# cp /etc/passwd ./
[root@web01 datas]# ls

passwd  s.txt


[root@web01 datas]# cat passwd 

root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
```

###### 搜索passwd文件以root关键字开头的所有行，并输出该行的第7列

```
[root@web01 datas]# awk -F : '/^root/{print $7}' passwd
/bin/bash

```


###### 搜索passwd文件以root关键字开头的所有行，并输出该行的第1列和第7列，中间以“，”号分割

```
[root@web01 datas]# awk -F : '/^root/{print $1 "," $7}' passwd 
root,/bin/bash

```

###### 只显示passwd的第一列和第七列，以逗号分割，且在所有行前面添加列名user，shell在最后一行添加"jx，/bin/qy"

```
[root@web01 datas]# awk -F : 'BEGIN {print "user,shell"} {print $1 "," $7} END{print "jx,/bin/qy"}' passwd 
user,shell
root,/bin/bash
bin,/sbin/nologin
daemon,/sbin/nologin
adm,/sbin/nologin
lp,/sbin/nologin
sync,/bin/sync
shutdown,/sbin/shutdown
halt,/sbin/halt
mail,/sbin/nologin
operator,/sbin/nologin
games,/sbin/nologin
ftp,/sbin/nologin
nobody,/sbin/nologin
systemd-network,/sbin/nologin
dbus,/sbin/nologin
polkitd,/sbin/nologin
sshd,/sbin/nologin
postfix,/sbin/nologin
chrony,/sbin/nologin
jx,/bin/qy

```

==注意：BEGIN 在所有数据读取行之前执行；END 在所有数据执行之后执行==


###### 将passwd文件中的用户id增加数值1并输出(id在第三列)

```
[root@web01 datas]# awk -F : -v i=1 '{print $3+1 "," $1}' passwd 
1,root
2,bin
3,daemon
4,adm
5,lp
6,sync
7,shutdown
8,halt
9,mail
12,operator
13,games
15,ftp
100,nobody
193,systemd-network
82,dbus
1000,polkitd
75,sshd
90,postfix
999,chrony

```

##### awk的内置变量

变量 | 说明
---|---
FILENAME| 文件名
NR | 记录所在行数
NF | 每行的列数


##### 案例

###### 统计passwd文件名，每行的行号，每行的列数


```
[root@web01 datas]# awk -F : '{print "filename :" FILENAME ", 行号 : " NR ", 总列数 : " NF}' passwd 
filename :passwd, 行号 : 1, 总列数 : 7
filename :passwd, 行号 : 2, 总列数 : 7
filename :passwd, 行号 : 3, 总列数 : 7
filename :passwd, 行号 : 4, 总列数 : 7
filename :passwd, 行号 : 5, 总列数 : 7
filename :passwd, 行号 : 6, 总列数 : 7
filename :passwd, 行号 : 7, 总列数 : 7
filename :passwd, 行号 : 8, 总列数 : 7
filename :passwd, 行号 : 9, 总列数 : 7
filename :passwd, 行号 : 10, 总列数 : 7
filename :passwd, 行号 : 11, 总列数 : 7
filename :passwd, 行号 : 12, 总列数 : 7
filename :passwd, 行号 : 13, 总列数 : 7
filename :passwd, 行号 : 14, 总列数 : 7
filename :passwd, 行号 : 15, 总列数 : 7
filename :passwd, 行号 : 16, 总列数 : 7
filename :passwd, 行号 : 17, 总列数 : 7
filename :passwd, 行号 : 18, 总列数 : 7
filename :passwd, 行号 : 19, 总列数 : 7

```


# sort

sort命令是在Linux里非常有用，它将文件进行排序，并将排序结果标准输出

### 语法
sort(选项)(参数)

##### 选项

选项 | 说明
---|---
-n | 依照数值的大小排序
-f | 以相反的顺序来排序
-t | 设置排序时所用的分隔字符
-k | 指定需要排序的列


##### 参数

指定待排序的文件列表


### 案例

##### 数据准备

```
[root@web01 datas]# touch sort.txt
[root@web01 datas]# vim sort.txt 

1:a:adadf
2:b:dsdsd
20:c:fdf
3:bc:fdfafda



[root@web01 datas]# sort sort.txt -n -r -t : -k 1
20:c:fdf
3:bc:fdfafda
2:b:dsdsd
1:a:adadf

```
