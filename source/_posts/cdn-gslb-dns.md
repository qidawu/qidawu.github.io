title: "浅析基于 DNS 解析方式的 GSLB"
date: 2015-09-23 22:33:55
updated:
tags: CDN
---

DNS 提供[域名](https://en.wikipedia.org/wiki/Domain_name)到 IP 的解析服务，

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

域名系统（DNS）的每一级只知道直接下级的位置，而无法获得跨级的位置。这种机制虽然看似低效，却能够提供分布式、高容错的服务，避免让域名系统（DNS）成为一个集中式的单点系统。

# DNS 如何解析域名？

对某个域名发起第一次查询请求时，负责处理递归查询的**本地 DNS 服务器**要发送好几次查询：

![A DNS recursor consults three name servers to resolve the address www.wikipedia.org.](https://upload.wikimedia.org/wikipedia/commons/thumb/7/77/An_example_of_theoretical_DNS_recursion.svg/1024px-An_example_of_theoretical_DNS_recursion.svg.png)

域名解析的时候，从右至左，先查根域，再查顶级域，再查一级域名，最终定位到 IP 地址。`dig` 命令加 `+trace` 参数可以**追踪**整个域名解析流程，从中了解经过的每一级 nameserver，其结果（简化并整理后）如下：

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

下面分别讲解每一级 nameserver 的作用：

## 根域名（Root nameserver）

`.` 代表的[根域名服务器（Root nameserver）](https://en.wikipedia.org/wiki/Root_name_server)，是 DNS 中最高级别的[域名服务器（nameserver）](https://en.wikipedia.org/wiki/Name_server)，负责返回顶级域名的[权威域名服务器（authoritative nameserver）](https://en.wikipedia.org/wiki/Authoritative_name_server)的地址。

早期的域名必须以英文句号“`.`”结尾，当用户访问 `www.wikipedia.org` 的 HTTP 服务时必须在地址栏中输入：`http://www.wikipedia.org.`，这样 DNS 才能够进行域名解析。如今 DNS 服务器已经可以自动补上结尾的句号。

目前全球共有 13 组根域名服务器，以英文字母 A 到 M 依序编号，域名格式为“`字母.root-servers.net`”。编号相同的根域名服务器使用同一个 IP，504 台根域名服务器总共只使用 13 个 IP，因此可以抵抗针对其所进行的分布式拒绝服务攻击（DDoS）。这些根域名服务器的运行软件皆为 [BIND](https://en.wikipedia.org/wiki/BIND)、[NSD](https://en.wikipedia.org/wiki/NSD)。

## 顶级域名（Top-level domain）

[顶级域名（Top-level domain, TLD）](https://en.wikipedia.org/wiki/Top-level_domain)主要分为两类：

* [通用顶级域名（Generic top-level domain，gTLD）](http://zh.wikipedia.org/wiki/GTLD)，如：`.com`、`.net`、`.org`、`.gov`、… 本站 `.me` 属于一个新开放的通用顶级域名。
* [国家和地区顶级域名（Country code top-level domain，ccTLD）](http://zh.wikipedia.org/wiki/CcTLD)，一般用两个字母的国家或地区名缩写代称，如：`.cn`、`.jp`、…

顶级域名的数量仍在不断增长中，除了英文字母的域名，还新增了各种语系的域名，如中文域名。

## 一级域名（First-level domain）

组织或个人通过域名代理服务商（如 GoDaddy、万网）进行申请、注册的域名，根据需要还可以自行在一级域名下新增二级、三级等多级域名。

## 资源记录（Resource record）

[DNS 区域（DNS zone）](https://en.wikipedia.org/wiki/DNS_zone)是层级化（hierarchical）域名系统（DNS）的一个子集。一个 DNS zone 往往就是一个域名。而[区域文件（Zone file）](https://en.wikipedia.org/wiki/Zone_file)则是一个描述 DNS zone 的文本文件。它包含了域名和 IP 地址等资源之间的映射，以[资源记录（Resource recerd, RR）](https://en.wikipedia.org/wiki/Domain_Name_System#DNS_resource_records)的文本形式进行组织。

```
STTL ID
@ IN SOA @
```

# 谁来分配和管理 IP ？

[ICANN](https://en.wikipedia.org/wiki/ICANN) 主要负责域名的管理、 IP 地址的空间分配、顶级域名系统以及根域名服务器系统的管理。
五个 [RIR](https://en.wikipedia.org/wiki/Regional_Internet_registry) 机构负责维护 IP 地址的分配与登记。

# GSLB 横向对比

GSLB（Global Server Load Balance，全局负载均衡，负责流量调度）作为 CDN 系统架构中最核心的部分，有三种常见的实现方式，其对比如下：

|比较项|基于 DNS 解析方式|基于 HTTP 重定向方式|基于 IP 路由方式|
|---|---|---|---|
|性能|本地 DNS 服务器和用户终端 DNS 缓存能力使 GSLB 的负载得到有效分担|GSLB 处理压力大，容易成为系统性能的瓶颈|借助 IP 网络设备完成负载均衡，没有单点性能瓶颈|
|准确度|定位准确度取决于本地 DNS 覆盖范围，**本地 DNS 设置错误会造成定位不准确**|在对用户 IP 地址数据进行有效维护的前提下，定位准确且精度高|就近性调度准确，但对设备健康性等动态信息响应会有延迟|
|效率|效率约等于 DNS 系统本身处理效率|依靠服务器做处理，对硬件资源的要求高|效率约等于 IP 设备本身效率|
|扩展性|扩展性和通用性好|扩展性较差，需对各种应用协议进行定制开发|通用性好，但适用范围有限|
|商用性|在 Web 加速领域使用较多|国内流媒体 CDN 应用较多|尚无商用案例|

# DNS 是什么？

域名系统（英文：Domain Name System，缩写：DNS）是因特网的一项服务。它作为将域名和 IP 地址相互映射的一个分布式数据库，能够使人更方便的访问互联网。

# LDNS 是什么？

本地 DNS 服务器（英文：Local DNS Server，缩写：LDNS）是用户所在局域网或 ISP 网络中使用的域名服务器。当客户端在浏览器里请求 www.wikipedia.org 时，浏览器会首先向本地 DNS 服务器请求将 abc.com 解析成 IP 地址，本地 DNS 服务器再**代为**向整个 DNS 系统发起查询，直到找到解析结果。

# DNS 域名解析流程

DNS 的查询机制给使用它的互联网应用带来额外的时延，有时时延还比较大，为了解决问题，需要引入“缓存”机制。缓存是指 DNS 查询结果在本地 DNS 服务器中缓存，当其它主机向它发起查询请求时，它就直接向主机返回缓存中能够找到的结果，直到数据过期。

在基于 DNS 解析方式下无论采用何种工作方式，都会有一些请求不会到达 GSLB，这是 DNS 系统本身的缓存机制在起作用。当用户请求的域名在本地 DNS 或本机（客户端浏览器）得到了解析结果，这些请求就不会达到 GSLB。Cache 更新时间越短，用户请求达到 GSLB 的几率越大。由于 DNS 的缓存机制屏蔽掉相当一部分用户请求，从而大大减轻了 GSLB 处理压力，使得系统抗流量冲击能力显著提升，这也是很多商业 CDN 选择 DNS 机制做全局负载均衡的原因之一。但弊端在于，如果在 DNS 缓存刷新间隔之内系统发生影响用户服务的变化，比如某个节点故障，某个链路拥塞等，用户依然会被调度到故障部位去。
