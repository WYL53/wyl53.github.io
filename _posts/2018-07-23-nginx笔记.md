# nginx笔记

nginx做反向代理的时候，代理的地址后面加路径的话，转发到后面的主机的地址会在设置的路径基础上加上请求地址去掉匹配的部分。
请求客户端请求http://localhost/abc/123/qqq时。
```
server {
  listen       80;
  server_name  localhost;
  #配置一
  location /abc{
  	proxy_pass http://api.host.com;
  }
}
```
后端服务器回收到：http://api.host.com/abc/123/qqq这样的请求。

```
...
  #配置二
  location /abc/{
  	proxy_pass http://api.host.com/;
  }
...
```
后端服务器回收到：http://api.host.com/123/qqq这样的请求。

