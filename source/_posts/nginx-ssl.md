---
title: Nginx 加密通信（HTTPS）
date: 2017-05-07 15:17:37
updated:
tags: Nginx
---

Nginx 中配置 HTTPS/SSL 加密是非常简单的，只需要将可选 HTTP 模块中的 `ngx_http_ssl_module` 编译进去即可。然后有两种方式开启 SSL 模式：

- `ssl on` 
- `listen 443 ssl` 此端口上接收的所有连接都工作在 SSL 模式。

建议使用 `listen` 指令的 `ssl` 参数替代 `ssl on` 指令，这样可以为同时处理 HTTP 和 HTTPS 请求的服务器提供更加紧凑的配置：

```
# http 和 https(ssl) 并存配置：

http {
    server {
        server_name example.com;
        listen 80;
        listen 443 ssl;
        
        ssl_certificate      example.com.crt;
        ssl_certificate_key  example.com.key;
        
        location / {
            proxy_pass http://127.0.0.1:81/;
        }
    }
}
```

注意，如果两个配置同时启用，HTTP 访问可能会报错：

```
400 Bad Request
The plain HTTP request was sent to HTTPS port
```

