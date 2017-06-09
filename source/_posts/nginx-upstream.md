---
title: Nginx 负载均衡
date: 2017-05-13 15:17:37
updated:
tags: Nginx
---

# 集群

Nginx 标准 HTTP 模块 [ngx_http_upstream_module](http://nginx.org/en/docs/http/ngx_http_upstream_module.html) 内置了集群和负载均衡功能，使用其中的 `upstream` 配合 `proxy_pass` 指令即可快速实现一个集群：

```
http {
    upstream backend {
        server backend1.example.com       weight=5;
        server 127.0.0.1:8080             max_fails=3 fail_timeout=30s;
        server unix:/tmp/backend3;

        server backup1.example.com:8080   backup;
        server backup2.example.com:8080   down;
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
```

其中 `server` 指令的参数意义如下：

| 参数                | 描述                                 |
| ----------------- | ---------------------------------- |
| weight=number     | 设置服务器的轮询权重，默认为 1。用于后端服务器性能不均的情况。   |
| max_conns=number  | 设置被代理服务器的最大可用并发连接数限制，默认为 0，表示没有限制。 |
| max_fails=number  | 设置最大失败重试次数，默认为 1。设置为 0 表示禁用重试。     |
| fail_timeout=time | 设置失败时间，默认 10 秒。                    |
| backup            | 将服务器标记为备份服务器。当主服务器不可用时，它将被传递请求。    |
| down              | 将服务器标记为永久不可用。                      |

# 负载均衡

负载均衡（Load Balance），其意思就是将运算或存储负载按一定的算法分摊到多个运算或存储单元上，下面介绍 Nginx 三种常见的负载均衡方法：

## ip hash

使用 Nginx `ip_hash` 指令，配置如下：

```
upstream backend {
    ip_hash;

    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
    server backend4.example.com;
}
```

`ip_hash` 指令指定集群使用基于**客户端 IP 地址**的负载均衡方法。 客户端 IPv4 地址的前三个八位字节或整个 IPv6 地址用作哈希键。 该方法确保来自同一客户端的请求将始终传递到同一台服务器，除非此服务器不可用，客户端请求则将被**转发**到另一台服务器（多数情况下，始终是同一台服务器）。
如果其中一台服务器需要临时删除，则应使用 `down` 参数标记，以便保留当前客户端 IP 地址的哈希值。

## 一致性 hash

使用 Nginx `hash` 指令，配置如下：

```
upstream backend {
    hash $remote_addr consistent;

    server backend1.example.com;
    server backend2.example.com;
}
```

`hash` 指令指定集群使用基于**指定 hash 散列键**的负载均衡方法。散列键可以包含文本，变量及其组合。请注意，从集群中添加或删除服务器可能会导致大量键被重新映射到不同的服务器。

解决办法是使用 `consistent` 参数启用  [ketama](http://www.last.fm/user/RJ/journal/2007/04/10/392555/) 一致性 hash 算法。 该算法将每个 server 虚拟成 n 个节点，均匀分布到 hash 环上。每次请求，根据配置的参数计算出一个 hash 值，在 hash 环上查找离这个 hash 最近的虚拟节点，对应的 server 作为该次请求的后端服务器。该算法确保在添加或删除服务器时，只会有少量键被重新映射到不同的服务器。这有助于为缓存服务器实现更高的缓存命中率。

## sticky cookie

会话保持是负载均衡的一个基本功能，为了确保与某个客户相关的所有应用请求能够由同一台服务器进行处理，我们需要在负载均衡上启用会话保持功能，以确保负载均衡的部署不会影响到正常的业务处理。而基于源地址的会话保持的问题在于，当多个客户是通过代理或地址转换的方式来访问服务器时，由于都分配到同一台服务器上，会导致服务器之间的负载失衡。

通过 cookie 实现客户端与后端服务器的会话保持，在一定条件下可以保证同一个客户端访问的都是同一个后端服务器。使用 Nginx `sticky` 指令，配置如下：

```
upstream backend {
    server backend1.example.com;
    server backend2.example.com;

    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```

下面这份配置在同一域名下有两个 location，分别对应了两组集群服务。为了分别实现会话保持，将 cookie 写入了对应的 path 下，避免 cookie 互相干扰，也减少了数据传输量：

```
http {
    upstream backend1 {
        server backup1.example.com:8080;
        server backup1.example.com:8081;
    	sticky cookie srv_backend1 path=/backend1;
    }

    upstream backend2 {
        server backup2.example.com:8080;
        server backup2.example.com:8081;
    	sticky cookie srv_backend2 path=/backend2;
    }
    
    server {
        server_name example.com;
        listen 80;
    
        location /backend1/ {
            proxy_pass http://backend1;
        }
        
        location /backend2/ {
            proxy_pass http://backend2;
        }
    }
}
```

# 健康检查

健康检查（Health Check）是保障集群可用性的重要手段，有三种常见的健康检查方法：

* 使用社区版 Nginx 的 `max_fails` 和 `fail_timeout` 指令进行被动式检查，不推荐使用，详见：《[nginx中健康检查(health_check)机制深入分析](https://segmentfault.com/a/1190000002446630)》；
* 使用[商业版 Nginx Plus](http://nginx.com/products/) 进行主动式检查，缺点是要收费；
* 使用 Nginx 第三方模块编译，例如：[nginx_upstream_check_module](https://github.com/yaoweibin/nginx_upstream_check_module) ；
* 使用 [Tengine](http://tengine.taobao.org/) 内置的[主动式健康检查](http://tengine.taobao.org/document_cn/http_upstream_check_cn.html)功能（该内置模块等同于第 3 点）。

## 主动式健康检查

以 `nginx_upstream_check_module` 第三方模块为例，演示配置如下：

```
upstream backend1 {
    check interval=3000 rise=2 fall=5 timeout=1000 type=http;
    check_keepalive_requests 100;
    check_http_send "HEAD /m/monitor.html HTTP/1.1\r\nConnection: keep-alive\r\nHost: check.com\r\n\r\n";
    check_http_expect_alive http_2xx http_3xx;
}
```

这段配置表示：

1. `check` 指令配置：每隔 `interval` 毫秒主动发送一个 `http` 健康检查包给后端服务器。请求超时时间为 `timeout` 毫秒。如果连续失败次数达到 `fall_count`，服务器就被认为是 down；如果连续成功次数达到 `rise_count`，服务器就被认为是 up。
2. `check_keepalive_requests` 指令配置：一个连接发送的请求数。
3. `check_http_send` 指令配置：请求包的内容（注意，这里必须[配置 `Host` 请求头否则可能报错](https://my.oschina.net/liuleidefeng/blog/786739)）。
4. `check_http_expect_alive` 指令配置：响应状态码为 `2XX` 和 `3XX` 表示请求成功、服务健康。

查看 Tomcat access.log 如下：

```
127.0.0.1 - - [06/Jun/2017:21:03:30 +0800] "HEAD /m/monitor.html HTTP/1.1" 200 -
127.0.0.1 - - [06/Jun/2017:21:03:33 +0800] "HEAD /m/monitor.html HTTP/1.1" 200 -
127.0.0.1 - - [06/Jun/2017:21:03:36 +0800] "HEAD /m/monitor.html HTTP/1.1" 200 -
127.0.0.1 - - [06/Jun/2017:21:03:39 +0800] "HEAD /m/monitor.html HTTP/1.1" 200 -
127.0.0.1 - - [06/Jun/2017:21:03:42 +0800] "HEAD /m/monitor.html HTTP/1.1" 200 -
127.0.0.1 - - [06/Jun/2017:21:03:45 +0800] "HEAD /m/monitor.html HTTP/1.1" 200 -
```

此时关闭某台后端服务器，一段时间后再访问，请求会被路由到其它服务器；重启后，该服务器自动加入集群。通过健康状态页面 `/status` 可见：

```
Nginx http upstream check status

Check upstream server number: 2, generation: 2

Index	Upstream	Name	Status	Rise counts	Fall counts	Check type	Check port
0	backend1	127.0.0.1:8080	up	4741	0	http	0
1	backend1	127.0.0.1:8081	down	0	2340	http	0
```

# 常用变量

## $upstream_addr

该模块中很常用的一个变量，用于标识集群中服务器的 IP 和端口。一般会加入到 Nginx 日志、同时脱敏后加入到响应头中，用于排查问题来源：

```
http {

    ...

    log_format  main  '"$http_x_forwarded_for" - "$upstream_addr" - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" $remote_addr $request_time_msec'
    access_log  logs/access.log  main;

    map $upstream_addr $short_address {
        ~^\d+\.\d+\.\d+\.(.*) '';
    }

    server {
        server_name example.com;
        listen 80;
        
        upstream backend {
            server 127.0.0.1:81;
            server 127.0.0.1:82;
        }
        
        location / {
            add_header X-From $short_address$1;
            proxy_pass http://backend/;
        }
    }
}
```

