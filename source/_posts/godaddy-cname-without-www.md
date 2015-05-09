title: "GoDaddy 中配置无 www 前缀域名"
date: 2015-04-12 23:19:55
updated: 
tags: DNS
---

搭好博客后，在权威域名服务商 GoDaddy 注册了一个新域名使用。当配置无 www 前缀域名时，却遇到了一些配置问题。

# 为什么要配置无 www 前缀的域名？

引述[知乎](http://www.zhihu.com/question/20064691)：

> www 是一个二级域名。无 www 才是一级域名。
>
> 拿掉它的原因有两个：
>
> 1. 遵循 Google 的关于域名长度的建议。
> 2. 保证网站的某一个页面有且只有一个唯一的 URL 可以访问。

# CNAME 配置问题

@ 记录指没有主机名的域名部份，如 foo.com 。于是想通过 GoDaddy 的 DNS ZONE FILE 配置一条 Host 为 @ 的 CNAME 记录：

|Host|Points To|TTL|
|---|---|---|
|www|cynthia903.github.io|1 Hour|
|@|cynthia903.github.io|1 Hour|

保存修改时，却报错了：

> THESE RECORDS WERE NOT PROCESSED DUE TO ERRORS:
> 
> The specified record already exists.
> 
> CNAME - @

原来按 [rfc1034](http://www.ietf.org/rfc/rfc1034.txt) 规定，是不能对 @ 记录做 CNAME 的，只能做 A 记录。

# 301 重定向跳转

经查询，常见做法是将 foo.com 做 301 重定向跳转到 www.foo.com 。GoDaddy 的 Forwarding 配置可以实现该功能：

CNBLOG.ME

|Forward to|Redirect|Type|
|---|---|---|
|http://www.cnblog.me|301 (Permanent)|Forward only|

# 解析测试

## 301 重定向跳转测试

使用 chrome 的开发者工具（F12 或 Ctrl+Shift+I 打开），访问 cnblog.me

|Path|Method|Status|
|---|---|---|
|http://cnblog.me/|GET|301 Moved Permanently|
|http://www.cnblog.me/|GET|200 OK| 

## DNS 解析测试

### Linux

使用 dig 命令测试，可见 www.cnblog.me 已经成功 CNAME 到 cynthia903.github.io 。

```
$ dig -t cname www.cnblog.me

;; QUESTION SECTION:
;www.cnblog.me.                 IN      CNAME

;; ANSWER SECTION:
www.cnblog.me.          3277    IN      CNAME   cynthia903.github.io.
```

### Windows

使用 nslookup 命令测试：

```
$ nslookup www.cnblog.me
服务器:  cache-b.guangzhou.gd.cn
Address:  202.96.128.166	// 当前使用的 DNS 服务器。如果无法解析，可以换用谷歌的试试（nslookup www.cnblog.me 8.8.8.8）

非权威应答:
名称:    github.map.fastly.net
Address:  199.27.78.133
Aliases:  www.cnblog.me
          cynthia903.github.io
```

# 参考

* 《[Configuring “@” CNAME record in GoDaddy control panel](http://serverfault.com/questions/486406/configuring-cname-record-in-godaddy-control-panel)》