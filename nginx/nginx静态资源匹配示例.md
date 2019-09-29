

### 示例1

匹配任何以 /css/ 开头的地址，匹配符合以后，还要继续往下搜索
只有后面的正则表达式没有匹配到时，这一条才会采用这一条

```
location  /css/ {
	root  H:/mytest/nginx-1.14.2/static;
}
```
如果输入http://host:ip/css 进入匹配

如果输入http://host:ip/123/css 不会进入匹配


### 示例2

匹配任何以 /css/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条

```
location ^~  /css/ {
	root  H:/mytest/nginx-1.14.2/static;
}
```

### 示例3
匹配所有以 gif,jpg或jpeg 结尾的请求

```
location ~* \.(gif|jpg|jpeg)$ {
   root  H:/mytest/nginx-1.14.2/static;
}
```

