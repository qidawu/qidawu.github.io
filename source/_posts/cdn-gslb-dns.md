title: "浅析基于 DNS 解析方式的 GSLB"
date: 2015-09-23 22:33:55
updated:
tags: CDN
---

## DNS 如何解析域名？

首先，通过一张图来了解 DNS 如何解析域名：

![A DNS recursor consults three name servers to resolve the address www.wikipedia.org.](https://upload.wikimedia.org/wikipedia/commons/thumb/7/77/An_example_of_theoretical_DNS_recursion.svg/1024px-An_example_of_theoretical_DNS_recursion.svg.png)

对某个域名发起第一次查询请求时，负责处理递归查询的 DNS 服务器要发送好几次查询。这是因为[域名系统](https://en.wikipedia.org/wiki/Domain_Name_System)是一个多层级、分布式的系统，每一级只知道直接下级的位置，而无法获得跨级的位置。这种机制虽然看似低效，但能够提供分布式、高容错的服务，避免让域名系统成为一个集中式的单点系统。

域名解析的时候，从右至左，先查 .root，再查 TLD，再查[权威 nameserver](https://en.wikipedia.org/wiki/Authoritative_name_server)，最终定位到 IP 地址。域名每一级都有独立的一个或多个 nameserver：

```
                    +---+                                 
                    | . |   root nameserver               
                    +-+-+                                 
                      |                                   
  +-------+-------+---+---+-------+-------+               
  |       |       |       |       |       |               
+-+-+   +-+-+   +-+-+   +-+-+   +-+--+  +-+-+   top-level 
|com|   |net|   |org|   |cn |   |gov |  |...|   domain
+---+   +---+   +-+-+   +---+   +----+  +---+             
                  |                                       
             +----+----+   authoritative                  
             |wikipedia|   nameserver                     
             +----+----+                                  
                  |                                       
          +---------------+                               
          |       |       |                               
        +-+-+   +-+-+   +-+--+                            
        |www|   |ftp|   |mail|   resource record          
        +---+   +---+   +----+                            
                                                          
      dig +trace www.wikipedia.org.                       
```

# 谁来分配和管理 IP ？

[ICANN](https://en.wikipedia.org/wiki/ICANN) 主要负责域名的管理、 IP 地址的空间分配、[通用顶级域名（Generic top-level domain，gTLD）](http://zh.wikipedia.org/wiki/GTLD)以及[国家和地区顶级域名（country code top-level domain，ccTLD）](http://zh.wikipedia.org/wiki/CcTLD)系统的管理、以及根服务器系统的管理。
五个 [RIR](https://en.wikipedia.org/wiki/Regional_Internet_registry) 机构负责维护 IP 地址的分配与登记。

## GSLB 横向对比

GSLB（Global Server Load Balance，全局负载均衡，负责流量调度）作为 CDN 系统架构中最核心的部分，有三种常见的实现方式，其对比如下：

|比较项|基于 DNS 解析方式|基于 HTTP 重定向方式|基于 IP 路由方式|
|---|---|---|---|
|性能|本地 DNS 服务器和用户终端 DNS 缓存能力使 GSLB 的负载得到有效分担|GSLB 处理压力大，容易成为系统性能的瓶颈|借助 IP 网络设备完成负载均衡，没有单点性能瓶颈|
|准确度|定位准确度取决于本地 DNS 覆盖范围，**本地 DNS 设置错误会造成定位不准确**|在对用户 IP 地址数据进行有效维护的前提下，定位准确且精度高|就近性调度准确，但对设备健康性等动态信息响应会有延迟|
|效率|效率约等于 DNS 系统本身处理效率|依靠服务器做处理，对硬件资源的要求高|效率约等于 IP 设备本身效率|
|扩展性|扩展性和通用性好|扩展性较差，需对各种应用协议进行定制开发|通用性好，但适用范围有限|
|商用性|在 Web 加速领域使用较多|国内流媒体 CDN 应用较多|尚无商用案例|

## DNS 是什么？

域名系统（英文：Domain Name System，缩写：DNS）是因特网的一项服务。它作为将域名和 IP 地址相互映射的一个分布式数据库，能够使人更方便的访问互联网。

## LDNS 是什么？

本地 DNS 服务器（英文：Local DNS Server，缩写：LDNS）是用户所在局域网或 ISP 网络中使用的域名服务器。当客户端在浏览器里请求 www.wikipedia.org 时，浏览器会首先向本地 DNS 服务器请求将 abc.com 解析成 IP 地址，本地 DNS 服务器再**代为**向整个 DNS 系统发起查询，直到找到解析结果。

## DNS 域名解析流程

DNS 的查询机制给使用它的互联网应用带来额外的时延，有时时延还比较大，为了解决问题，需要引入“缓存”机制。缓存是指 DNS 查询结果在本地 DNS 服务器中缓存，当其它主机向它发起查询请求时，它就直接向主机返回缓存中能够找到的结果，直到数据过期。

在基于 DNS 解析方式下无论采用何种工作方式，都会有一些请求不会到达 GSLB，这是 DNS 系统本身的缓存机制在起作用。当用户请求的域名在本地 DNS 或本机（客户端浏览器）得到了解析结果，这些请求就不会达到 GSLB。Cache 更新时间越短，用户请求达到 GSLB 的几率越大。由于 DNS 的缓存机制屏蔽掉相当一部分用户请求，从而大大减轻了 GSLB 处理压力，使得系统抗流量冲击能力显著提升，这也是很多商业 CDN 选择 DNS 机制做全局负载均衡的原因之一。但弊端在于，如果在 DNS 缓存刷新间隔之内系统发生影响用户服务的变化，比如某个节点故障，某个链路拥塞等，用户依然会被调度到故障部位去。
