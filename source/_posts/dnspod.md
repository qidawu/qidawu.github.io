title: "使用 DNSPod 解决 GoDaddy 域名解析不稳定的问题"
date: 2015-04-26 16:12:25
updated: 
tags: DNS
description: "由于众所周知的原因，GoDaddy 的 Nameservers 有时会被墙，导致其在国内的域名解析服务一直都很不稳定，因此抽空改用了国内的 DNSPod 服务，步骤如下。"
---

由于众所周知的原因，GoDaddy 的 Nameservers 有时会被墙，导致其在国内的域名解析服务一直都很不稳定，因此抽空改用了国内的 DNSPod 服务，步骤如下。

# 注册 DNSPod 账号

DNSPod 已并入腾讯，使用 QQ 账号绑定并登录即可。

# 添加域名

添加在域名服务商中购买的域名即可，不带 www 前缀。

# 添加记录

这里各人需求不同，设置因人而异。

对于我的需求来说，需要为我购买的域名添加两条 CNAME 记录，分别是带 www 前缀和不带的：

|主机记录|记录类型|记录值|
|---|---|---|
|www|CNAME|cynthia903.github.io|
|@|CNAME|cynthia903.github.io|

其中，DNSPod 的备注清晰地描述道，主机记录就是域名前缀，常见用法有：

|主机记录|描述|
|---|---|
|www|解析后的域名为 www.cnblog.me|
|@|直接解析主域名 cnblog.me|
|*|泛解析，匹配其他所有域名 *.cnblog.me|

# 修改 Nameservers

什么是 Nameservers ？

> Your nameservers determine where your domain points based on whether you are parking, forwarding, or hosting your domain with us or with another registrar.

什么是 NS 记录？

> 即域名服务器记录，用于指定该域名由哪个 DNS 服务器进行解析。注册域名时，总有默认的 DNS 服务器。

例如 DNSPod 默认配置并提供的 NS 记录如下：

|主机记录|记录类型|记录值|
|---|---|---|
|@|NS|f1g1ns1.dnspod.net|
|@|NS|f1g1ns2.dnspod.net|

## 使用 DNSPod 的域名解析服务

进入 GoDaddy 的 NAMESERVER SETTINGS ，将 SETUP TYPE 从 Standard 切换到 Custom ，并添加 DNSPod 提供的 NS 记录：

```
f1g1ns1.dnspod.net
f1g1ns2.dnspod.net
```

# DNS 解析测试

使用 dig 命令测试，可见 www.cnblog.me 的 NS 记录已经配置成功：

```
$ dig +trace www.cnblog.me

......（此处略过 Root、TLD，只展示权威域名服务器的解析结果）
www.cnblog.me.          600     IN      CNAME   cynthia903.github.io.
cnblog.me.              600     IN      NS      f1g1ns1.dnspod.net.
cnblog.me.              600     IN      NS      f1g1ns2.dnspod.net.
```