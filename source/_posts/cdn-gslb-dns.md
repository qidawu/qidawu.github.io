---
title: "浅析基于 DNS 解析方式的 GSLB"
date: 2015-09-30 22:33:55
updated:
tags: 计算机网络
---

GSLB（Global Server Load Balance，全局负载均衡）作为 CDN 系统架构中最核心的部分，负责流量调度。本文站在服务提供方的视角，做一些技术总结。

# GSLB 横向对比

下表是三种常见的实现方式对比：

| 比较项  | 基于 DNS 解析方式                              | 基于 HTTP 重定向方式                   | 基于 IP 路由方式                 |
| ---- | ---------------------------------------- | ------------------------------- | -------------------------- |
| 性能   | 本地 DNS 服务器和用户终端 DNS 缓存能力使 GSLB 的负载得到有效分担 | GSLB 处理压力大，容易成为系统性能的瓶颈          | 借助 IP 网络设备完成负载均衡，没有单点性能瓶颈  |
| 准确度  | 定位准确度取决于本地 DNS 覆盖范围，用户的**本地 DNS 设置错误会造成定位不准确** | 在对用户 IP 地址数据进行有效维护的前提下，定位准确且精度高 | 就近性调度准确，但对设备健康性等动态信息响应会有延迟 |
| 效率   | 效率约等于 DNS 系统本身处理效率                       | 依靠服务器做处理，对硬件资源的要求高              | 效率约等于 IP 设备本身效率            |
| 扩展性  | 扩展性和通用性好                                 | 扩展性较差，需对各种应用协议进行定制开发            | 通用性好，但适用范围有限               |
| 商用性  | 在 Web 加速领域使用较多                           | 国内流媒体 CDN 应用较多                  | 尚无商用案例                     |

其中，基于 DNS 解析方式的 GSLB 有两个注意点：

## 准确度

本地 DNS 服务器（英文：Local DNS Server，缩写：LDNS）是用户所在局域网或 ISP 网络中使用的域名服务器，定位准确度就取决于它了。因为当用户在浏览器里访问某个域名时，浏览器会首先向 LDNS 发起查询，LDNS 再**代为**向整个 DNS 域名系统发起查询，直到找到解析结果。域名解析流程详见[本文](http://www.cnblog.me/2015/09/24/dns/)。

如果 LDNS 设置不当，例如没有使用当前 ISP 提供的当地 LDNS，如 8.8.8.8，这种实现方式可能会误判用户的位置，从而将用户误导到错误的 CDN 缓存节点，造成加速效果差的问题。

## 缓存

DNS 的查询机制给使用它的互联网应用带来额外的时延，有时时延还比较大，为了解决问题，引入了“缓存”机制。缓存是指 DNS 查询结果在 LDNS 中缓存，当其它主机向它发起查询请求时，它就直接向主机返回缓存中能够找到的结果，直到数据过期。

在基于 DNS 解析方式下无论采用何种工作方式，都会有一些请求不会到达 GSLB，这是 DNS 系统本身的缓存机制在起作用。当用户请求的域名在本地 DNS 或本机（客户端浏览器）得到了解析结果，这些请求就不会达到 GSLB。Cache 更新时间越短，用户请求到达 GSLB 的几率越大。由于 DNS 的缓存机制屏蔽掉相当一部分用户请求，从而大大减轻了 GSLB 处理压力，使得系统抗流量冲击能力显著提升，这也是很多商业 CDN 选择 DNS 机制做全局负载均衡的原因之一。但弊端在于，如果在 DNS 缓存刷新间隔之内系统发生影响用户服务的变化，比如某个节点故障，某个链路拥塞等，用户依然会被调度到故障点去。

# 智能 DNS 实现浅析

基于 DNS 解析方式的 GSLB 的实现关键，就在于使 DNS “智能化”。简单来说，就是通过建立 IP 地址访问列表，判断用户的访问来源，以确定其访问节点的位置。下面浅析如何实现智能 DNS：

## IP 地址收集策略

由于基于 DNS 解析方式的 CDN 使用 LDNS 进行寻址，因此我们只需要收集互联网上 DNS 服务器的 IP 地址。这样一来，收集的数量就会大大降低。为了更进一步缩小范围，一般使用 IP 地址加子网掩码的形式，如 123.175.0.0/16。在 IP 地址列表文件，就这么一行，却可以囊括很多 DNS 服务器。

## IP 地址收集方法

除了可以跟第三方购买 IP 地址段之外，这里重点介绍下如何自行收集 IP 地址段。

![ICANN](https://upload.wikimedia.org/wikipedia/en/thumb/4/4f/ICANN.svg/171px-ICANN.svg.png)

[ICANN](https://en.wikipedia.org/wiki/ICANN) —— 一个负责 IP 地址分配以及域名管理的机构，与之关联的五个 [RIR](https://en.wikipedia.org/wiki/Regional_Internet_registry) 机构负责替 ICANN 分配与登记部分区域的 IP 地址段：

| RIR                                | Region                                   |
| ---------------------------------- | ---------------------------------------- |
| [AFRINIC](http://www.afrinic.net/) | Africa region                            |
| [APNIC](http://www.apnic.net/)     | Asia and Pacific region                  |
| [ARIN](http://www.arin.net/)       | Canada, many Caribbean and North Atlantic islands, and the United States |
| [LACNIC](http://www.lacnic.net/)   | Latin America and parts of the Caribbean |
| [RIPE NCC](http://www.ripe.net/)   | Europe, the Middle East and parts of Central Asia |

可见，亚太地区的 IP 地址由 APNIC 分配，访问[这里](https://www.apnic.net/publications/research-and-insights/stats)可以知道在何处得到 IP 地址分配的有用信息。进入 FTP ，阅读 README 以了解该下载哪个文件以及文件的格式。下载 `delegated-apnic-latest` 文件，过滤出分配给中国大陆（CN）的 IP 地址。

然后可以通过 [CNNIC IP 地址注册信息查询系统](http://ipwhois.cnnic.cn/)查询这个地址段属于哪个运营商，但一次只能查询一个地址段，根本无法手工完成所有地址段的查询，因此推荐在 Linux 下使用 whois 工具以遍历的方式逐个查询，然后按关键字归类、去重、排序，按运营商产生几个独立的文件。如果各 IP 地址租用方未能按统一的标准在 APNIC 提交注册信息则需要特殊处理。

## IP 地址列表使用

最后，将每个 IP 地址列表文件关联一个 Bind 的视图 View。定义视图的目的在于，当有来自某个文件所列 IP 范围内的客户发起查询请求时，使用本视图的区文件进行域名解析。通俗的说，就是让某个运营商线路的用户，去访问某个运营商机房的服务器。