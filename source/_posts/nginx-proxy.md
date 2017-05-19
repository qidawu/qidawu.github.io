---
title: Nginx 反向代理
date: 2017-05-06 15:17:37
updated:
tags: Nginx
---

# 什么是代理？

一张图了解两种代理模式的区别：

![两种代理模式](/img/sa/proxy.jpg)

常见应用场景：

- 正向代理：VPN 翻墙
- 反向代理：Web 站点服务

# Nginx 反向代理

Nginx 代理功能由标准 HTTP 模块中内置的 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) 提供，因此无需额外编译，常见配置如下：

```
http {
    server {
        server_name example.com;
        listen 80;
        
        location / {
            proxy_pass http://127.0.0.1:81/;
        }
    }
}
```

`proxy_pass` 指令用于配置被代理服务器的协议和地址。除了配置单机，还可以配置集群，详见 [Nginx 负载均衡](/2017/05/13/nginx-upstream/) 。

# 解决代理后的问题

使用反向代理之后，上游服务器（如 Tomcat）无法获取真实的访问来源信息（如协议、域名、访问 IP），例如下面代码：

```java
request.getScheme() // 总是 http，而不是实际的 http 或 https
request.isSecure() // 总是 false
request.getRemoteAddr() // Nginx IP
request.getServerName() // 127.0.0.1
request.getRequestURL() // http://127.0.0.1:81/index
response.sendRedirect(...) // 总是重定向到 http
```

这个问题需要在 Nginx 和 Tomcat 中做一些配置以解决问题。

## Nginx 配置

使用 `proxy_set_header` 指令为上游服务器添加请求头：

```
http {
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    server {
        ...
    }
}
```

## Tomcat 配置

在 Tomcat 的 `server.xml` 中配置 [RemoteIpValve](http://tomcat.apache.org/tomcat-7.0-doc/api/org/apache/catalina/valves/RemoteIpValve.html) 让代码能够获取真实 IP 和协议：

```
<Valve className="org.apache.catalina.valves.RemoteIpValve" 
    remoteIpHeader="X-Forwarded-For" 
    protocolHeader="X-Forwarded-Proto" />
```

解决结果：

```java
request.getScheme() // 实际的 http 或 https
request.isSecure() // 对应的 false 或 true
request.getRemoteAddr() // 用户 IP
request.getHeader("X-Real-IP") // 用户 IP
request.getServerName() // example.com
request.getRequestURL() // 对应的 http://example.com/index 或 https://example.com/index
response.sendRedirect(...) // 实际的 http 或 https
```
