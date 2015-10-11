title: DNS 知识总结
date: 2015-09-24 19:31:09
updated: 2015-10-11 23:31:09
tags: DNS
---

工作、生活中常与域名打交道，尤其是近期需要为公司的系统上 CDN 服务，因此对 CDN 的核心 DNS 做了一番重温、总结。

# DNS 是什么？

DNS 是互联网的一项基础服务，它将人类易记的[域名](https://en.wikipedia.org/wiki/Domain_name)解析为不易记的 IP 地址，使人更方便的访问互联网。

# DNS 的结构？

[域名系统（DNS）](https://en.wikipedia.org/wiki/Domain_Name_System)是一个多层级、分布式的系统，就如同一个树状结构：

```
                    +---+                                 
                    | . |   Root nameserver               
                    +-+-+                                 
                      |                                   
  +-------+-------+---+---+-------+-------+               
  |       |       |       |       |       |               
+-+-+   +-+-+   +-+-+   +-+-+   +-+-+   +-+-+    
|com|   |net|   |org|   |gov|   |cn |   |...|   Top-level domain
+---+   +---+   +-+-+   +---+   +---+   +---+             
                  |                                       
             +----+----+                     
             |wikipedia|   First-level domain                     
             +----+----+                                  
                  |                                       
          +---------------+                               
          |       |       |                               
        +-+-+   +-+-+   +-+--+                            
        |www|   |ftp|   |mail|   Resource record          
        +---+   +---+   +----+                           
```

域名系统（DNS）的每一级只知道直接下级的位置，而无法获得跨级的位置，因此在域名解析的时候，需要自上而下、逐级查询。这种机制虽然看似低效，却能够提供分布式、高容错的服务，避免让域名系统（DNS）成为一个集中式的单点系统。

# DNS 如何解析域名？

对某个域名发起第一次解析请求时，负责处理递归查询的**本地 DNS 服务器**要发送好几次查询：

![A DNS recursor consults three name servers to resolve the address www.wikipedia.org.](https://upload.wikimedia.org/wikipedia/commons/thumb/7/77/An_example_of_theoretical_DNS_recursion.svg/1024px-An_example_of_theoretical_DNS_recursion.svg.png)

域名解析的时候，需要自上而下、逐级查询：先查根域，再查顶级域，再查一级域名，最终定位到 IP 地址。`dig` 命令加 `+trace` 参数可以**追踪**整个域名解析过程，从中了解经过的每一级 nameserver，其结果简化如下：

```                 
$ dig +trace www.wikipedia.org

;; global options: +cmd
.                    NS      a.root-servers.net.
.                    NS      b.root-servers.net.
.                    NS      c.root-servers.net.
.                    NS      d.root-servers.net.
.                    NS      e.root-servers.net.
.                    NS      f.root-servers.net.
.                    NS      g.root-servers.net.
.                    NS      h.root-servers.net.
.                    NS      i.root-servers.net.
.                    NS      j.root-servers.net.
.                    NS      k.root-servers.net.
.                    NS      l.root-servers.net.
.                    NS      m.root-servers.net.
;; Received 315 bytes from 202.96.128.166#53(202.96.128.166) in 642 ms

org.                 NS      a0.org.afilias-nst.info.
org.                 NS      a2.org.afilias-nst.info.
org.                 NS      b0.org.afilias-nst.org.
org.                 NS      b2.org.afilias-nst.org.
org.                 NS      c0.org.afilias-nst.info.
org.                 NS      d0.org.afilias-nst.org.
;; Received 691 bytes from 199.7.83.42#53(l.root-servers.net) in 408 ms

wikipedia.org.       NS      ns0.wikimedia.org.
wikipedia.org.       NS      ns1.wikimedia.org.
wikipedia.org.       NS      ns2.wikimedia.org.
;; Received 651 bytes from 199.19.53.1#53(c0.org.afilias-nst.info) in 1155 ms

www.wikipedia.org.   A       198.35.26.96
;; Received 90 bytes from 91.198.174.239#53(ns2.wikimedia.org) in 365 ms
```

下面分别介绍每一级 nameserver：

## 根域名（Root nameserver）

`.` 代表的[根域名服务器（Root nameserver）](https://en.wikipedia.org/wiki/Root_name_server)，是 DNS 中最高级别的[域名服务器（nameserver）](https://en.wikipedia.org/wiki/Name_server)，负责返回顶级域名的[权威域名服务器（authoritative nameserver）](https://en.wikipedia.org/wiki/Authoritative_name_server)的地址。

早期的域名必须以英文句号“`.`”结尾，当用户访问 `www.wikipedia.org` 的 HTTP 服务时必须在地址栏中输入：`http://www.wikipedia.org.`，这样 DNS 才能够进行域名解析。如今 DNS 服务器已经可以自动补上结尾的句号。

目前全球共有 13 组根域名服务器，以英文字母 A 到 M 依序编号，域名格式为“`字母.root-servers.net`”。编号相同的根域名服务器使用同一个 IP，数百台根域名服务器总共只使用 13 个 IP，因此可以抵抗针对其所进行的分布式拒绝服务攻击（DDoS）。这些根域名服务器的运行软件皆为 [BIND](https://en.wikipedia.org/wiki/BIND)、[NSD](https://en.wikipedia.org/wiki/NSD)。

## 顶级域名（Top-level domain）

[顶级域名（Top-level domain, TLD）](https://en.wikipedia.org/wiki/Top-level_domain)主要分为两类：

* [通用顶级域名（Generic top-level domain，gTLD）](http://zh.wikipedia.org/wiki/GTLD)，如常见的：`.com`、`.net`、`.org`、`.gov`、… 本站 `.me` 是一个新开放的通用顶级域名。
* [国家和地区顶级域名（Country code top-level domain，ccTLD）](http://zh.wikipedia.org/wiki/CcTLD)，一般用两个字母的国家或地区名缩写代称，如：`.cn`、`.jp`、…

顶级域名的数量仍在不断增长中，除了英文字母的域名，还不断新增各种语系的域名，如中文域名。

## 一级域名（First-level domain）

组织或个人通过域名代理服务商（如 GoDaddy、万网）进行注册的域名。根据需要还可以自行在一级域名下新增二级、三级等子域名。

## 资源记录（Resource record）

域名系统中，一般一个[域（DNS zone）](https://en.wikipedia.org/wiki/DNS_zone)通过一个 [zone 文件](https://en.wikipedia.org/wiki/Zone_file)保存该域的相关配置信息。zone 文件包含了域名和 IP 地址等资源之间的映射，以[资源记录（Resource recerd, RR）](https://en.wikipedia.org/wiki/Domain_Name_System#DNS_resource_records)的文本形式进行组织。[这里](https://en.wikipedia.org/wiki/List_of_DNS_record_types)列举了所有的资源记录类型。

以域名 `example.com` 为例，其 zone 文件简化如下：

|name|ttl|record class|record type|record data|comment|
|---|---|---|---|---|---|
|example.com.|1h|IN|NS|ns|ns.example.com is a nameserver for example.com|
|ns|1h|IN|A|192.0.2.2|IPv4 address for ns.example.com|
|example.com.|1h|IN|A|192.0.2.1|IPv4 address for example.com|
|www|1h|IN|CNAME|example.com.|www.example.com is an alias for example.com|

# 总结

综上，注册域名、管理资源记录都是站长最常见的操作。至于域名解析就交给本地 DNS 服务器代为处理，一般用户都无需操心。