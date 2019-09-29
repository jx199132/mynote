# nginx负载均衡
网站的访问量越来越大，服务器的服务模式也得进行相应的升级，比如分离出数据库服务器、分离出图片作为单独服务，这些是简单的数据的负载均衡，将压力分散到不同的机器上。有时候来自web前端的压力，也能让人十分头痛。怎样将同一个域名的访问分散到两台或更多的机器上呢？这其实就是另一种负载均衡了，nginx自身就可以做到，只需要做个简单的配置就行。

nginx不但可以作为强大的web服务器，也可以作为一个反向代理服务器，而且nginx还可以按照调度规则实现动态、静态页面的分离，可以按照轮询、ip哈希、URL哈希、权重等多种方式对后端服务器做负载均衡，同时还支持后端服务器的健康检查。

nginx 的 upstream目前支持几种种方式的分配 
- 1)、轮询（默认） 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。 
- 2)、weight 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 
- 3)、ip_hash 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。  


# nginx负载均衡配置,主要是proxy_pass,upstream的使用
在http段做如下配置，即可实现两个域名

```
upstream  www.linuxidc.com  
{
    server   10.0.1.50:8080;
    server   10.0.1.51:8080;
}
 
upstream  blog.linuxidc.com   
{
    server   10.0.1.50:8080;
    server   10.0.1.51:8080;
}
 
server
{
    listen  80;
    server_name  www.linuxidc.com;
 
    location / {
        proxy_pass        http://www.linuxidc.com;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        #跟代理服务器连接的超时时间，当一台服务器当掉时，过5秒转发到另外一台服务器
        proxy_connect_timeout 5;
    }
}
 
server
{
    listen  80;
    server_name  blog.linuxidc.com wode.linuxidc.com;
 
    location / {
        proxy_pass        http://www.linuxidc.com;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
```

