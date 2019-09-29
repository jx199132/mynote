### location
配置块 server
location会根据请求uri来匹配 与自身配置的uri进行匹配，如果可以匹配，就选择location块中的配置来处理用户请求.
语法：location [=|~|~*|^~|@]/uri

= ：把uri当做字符串，做完全匹配
~ ：表示匹配uri时大小写是敏感的
~*：表示匹配时忽略大小写敏感
^~：表示匹配uri时只需要前半部分满足uri即可 例如  ^~/images/     那么以 /images/开始的请求都会匹配上

### 文件路径定义
1  root方式设置资源路径
语法  root path
默认 root html
配置块 http 、server 、 location 、 if
例如 定义 资源相对于 http请求的目录
location /download/ {
  root /opt/web/html
}
如果请求uri是  /download/index/test.html ，那么 web服务器将会返回  /opt/web/html/download/index/test.html

2  alias 方式设置资源路径
配置块 location
例如 ： 若有有一个请求 uri 是 /conf/nginx.conf ，而 用户实际想访问的文件在 /usr/local/nginx/conf/nginx.conf 
 alias方式
    localtion /conf{
      alias /usr/local/nginx/conf;
   }
  root方式
   location /conf{
     root /usr/local/nginx;
  }
总结：root 用于定义 文件的根目录， 然后  根目录  + 请求的路径 进行拼接组合
          alias 在请求的uri 路径中 已经把/conf 这段字符串丢掉  最终 配置的目录 + uri替换掉/conf的字符串


### index
访问首页
配置块  http  、server 、 location

location / {
   root path;
   index /index.html /index.jsp /index.php
}
请求过来之后是域名path，这时候nginx会返回 path/index.html 中的结果，如果没有获取到 就继续 path/index.jsp

### error_page
配置块 http 、server、location、if
根据http返回码重定向到页面
例如  
error_page 404 /404.html
error_page 502 503 504 /50x.html
