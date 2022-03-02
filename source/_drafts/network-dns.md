---
title: 应用层 DNS 协议、相关命令总结
date: 2022-01-15 23:32:08
updated:
tags: 计算机网络
typora-root-url: ..
---

# 层次化域名空间

![DNS](/img/network/application-layer/dns/DNS.png)



|      | Domain types                          | DNS zone                                                     | Name server                                                  | Reference                                         |
| ---- | ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------- |
| 0    | Root domain<br>根域名                 | [DNS root zone](https://en.wikipedia.org/wiki/DNS_root_zone)<br>DNS 根区 | [Root name server](https://en.wikipedia.org/wiki/Root_name_server)<br>根域名服务器 | https://en.wikipedia.org/wiki/Root_domain         |
| 1    | Top-level domain (TLD)<br>顶级域名    | [DNS zone](https://en.wikipedia.org/wiki/DNS_zone)           | TLD name server<br>顶级域名服务器                            | https://en.wikipedia.org/wiki/Top-level_domain    |
| 2    | Second-level domain (SLD)<br>二级域名 | [DNS zone](https://en.wikipedia.org/wiki/DNS_zone)           | [Authoritative name server](https://en.wikipedia.org/wiki/Name_server#Authoritative_name_server)<br>权威域名服务器 | https://en.wikipedia.org/wiki/Second-level_domain |
| 3    | Sub domain<br>子域名                  |                                                              |                                                              | https://en.wikipedia.org/wiki/Subdomain           |

## 根域名（Root Domain）

> 由于 ICANN 管理着所有的顶级域名，所以它是最高一级的域名节点，被称为根域名（root domain）。在有些场合，`www.example.com` 被写成 `www.example.com.`，即最后还会多出一个点 `.`。这个点 `.` 就是根域名。
>

## 根域名服务器（Root Name Server）

https://www.iana.org/domains/root/servers

> 由于早期的 DNS 查询结果是一个 512 字节的 UDP 数据包。这个包最多可以容纳 13 个服务器的地址，因此就规定全世界有 13 个根域名服务器，编号从 `a.root-servers.net` 一直到 `m.root-servers.net`：
>
> | HOSTNAME           | IP ADDRESSES                      | OPERATOR                                                     |
>| ------------------ | --------------------------------- | ------------------------------------------------------------ |
> | a.root-servers.net | 198.41.0.4, 2001:503:ba3e::2:30   | Verisign, Inc.                                               |
> | b.root-servers.net | 199.9.14.201, 2001:500:200::b     | University of Southern California, Information Sciences Institute |
> | c.root-servers.net | 192.33.4.12, 2001:500:2::c        | Cogent Communications                                        |
> | d.root-servers.net | 199.7.91.13, 2001:500:2d::d       | University of Maryland                                       |
> | e.root-servers.net | 192.203.230.10, 2001:500:a8::e    | NASA (Ames Research Center)                                  |
> | f.root-servers.net | 192.5.5.241, 2001:500:2f::f       | Internet Systems Consortium, Inc.                            |
> | g.root-servers.net | 192.112.36.4, 2001:500:12::d0d    | US Department of Defense (NIC)                               |
> | h.root-servers.net | 198.97.190.53, 2001:500:1::53     | US Army (Research Lab)                                       |
> | i.root-servers.net | 192.36.148.17, 2001:7fe::53       | Netnod                                                       |
> | j.root-servers.net | 192.58.128.30, 2001:503:c27::2:30 | Verisign, Inc.                                               |
> | k.root-servers.net | 193.0.14.129, 2001:7fd::1         | RIPE NCC                                                     |
> | l.root-servers.net | 199.7.83.42, 2001:500:9f::42      | ICANN                                                        |
> | m.root-servers.net | 202.12.27.33, 2001:dc3::35        | WIDE Project                                                 |
> 
> 这 13 台根域名服务器由 12 个组织独立运营。其中，Verisign 公司管理两台根域名服务器：A 和 J。每家公司为了保证根域名服务器的可用性，会部署多个节点，比如单单 Verisign 一家公司就部署了 104 台根域名服务器（2016 年 1 月数据）。
>
> 所以，根域名服务器其实[不止 13 台](https://www.icann.org/news/blog/there-are-not-13-root-servers)。据统计，截止 2016 年 1 月，全世界共有 517 台根域名服务器。你可以在 https://root-servers.org/ 这个网站查到所有根域名服务器的信息。

## 根提示文件（Root Hints File）

https://www.internic.net/domain/named.root

> 根域名服务器虽然有域名，但是最少必须知道一台的 IP 地址，否则就会陷入循环查询。一般来说，本机都保存一份根域名服务器的 IP 地址的缓存，叫做 [named.root](https://www.internic.net/domain/named.root) 文件：
>
> ```
> .                        3600000      NS    A.ROOT-SERVERS.NET.
> A.ROOT-SERVERS.NET.      3600000      A     198.41.0.4
> A.ROOT-SERVERS.NET.      3600000      AAAA  2001:503:ba3e::2:30
> 
> .                        3600000      NS    B.ROOT-SERVERS.NET.
> B.ROOT-SERVERS.NET.      3600000      A     199.9.14.201
> B.ROOT-SERVERS.NET.      3600000      AAAA  2001:500:200::b
> 
> ...
> ```
>
> 这个文件记录了 13 台根域名服务器的 IP 地址。

参考：[4.1 Address resolution mechanism](https://en.wikipedia.org/wiki/Domain_Name_System#Address_resolution_mechanism)

> For proper operation of its domain name resolver, a network host is configured with an initial cache (*hints*) of the known addresses of the root name servers. The hints are updated periodically by an administrator by retrieving a dataset from a reliable source.

域名解析的**循环依赖**问题及其解决方案：[4.3 Circular dependencies and glue records](https://en.wikipedia.org/wiki/Domain_Name_System#Circular_dependencies_and_glue_records)

> Name servers in delegations are identified by name, rather than by IP address. This means that a resolving name server must issue another DNS request to find out the IP address of the server to which it has been referred. If the name given in the delegation is a subdomain of the domain for which the delegation is being provided, there is a [circular dependency](https://en.wikipedia.org/wiki/Circular_dependency).
>
> In this case, the name server providing the delegation must also provide one or more IP addresses for the authoritative name server mentioned in the delegation. This information is called <u>*glue*</u>. The delegating name server provides this glue in the form of records in the <u>*additional section*</u> of the DNS response, and provides the delegation in the <u>*authority section*</u> of the response. A glue record is a combination of the name server and IP address.
>
> For example, if the [authoritative name server](https://en.wikipedia.org/wiki/Authoritative_name_server) for `example.org` is `ns1.example.org`, a computer trying to resolve `www.example.org` first resolves `ns1.example.org`. As `ns1` is contained in `example.org`, this requires resolving `example.org` first, which presents a circular dependency. To break the dependency, the name server for the [top level domain](https://en.wikipedia.org/wiki/Top_level_domain) org includes glue along with the delegation for `example.org`. The glue records are address records that provide IP addresses for `ns1.example.org`. The resolver uses one or more of these IP addresses to query one of the domain's authoritative servers, which allows it to complete the DNS query.

## 根区文件（Root Zone File）

https://www.internic.net/domain/root.zone

> DNS 根域名服务器（Root name server）负责维护 DNS 根区文件（Root zone file）。该文件保存了所有顶级域名（TLD）的托管信息，所以非常大，超过 2MB。
>
> 举例来说，顶级域名 `.com` 可以查到 13 个域名服务器：
>
> ```bash
> com.            172800  IN  NS  a.gtld-servers.net.
> com.            172800  IN  NS  b.gtld-servers.net.
> com.            172800  IN  NS  c.gtld-servers.net.
> com.            172800  IN  NS  d.gtld-servers.net.
> com.            172800  IN  NS  e.gtld-servers.net.
> com.            172800  IN  NS  f.gtld-servers.net.
> com.            172800  IN  NS  g.gtld-servers.net.
> com.            172800  IN  NS  h.gtld-servers.net.
> com.            172800  IN  NS  i.gtld-servers.net.
> com.            172800  IN  NS  j.gtld-servers.net.
> com.            172800  IN  NS  k.gtld-servers.net.
> com.            172800  IN  NS  l.gtld-servers.net.
> com.            172800  IN  NS  m.gtld-servers.net.
> ```
>
> 也就是说，`.com` 域名的解析结果，可以到这个 13 个服务器的任一台查询。细心的读者可能发现，这些服务器本身也是使用域名（比如 `a.gtld-servers.net.`）标识，那么还得去查询它们指向的服务器，这样很容易造成循环查询。
>
> 因此，DNS 根区文件还会同时提供这些服务器的 IP 地址（IPv4 和 IPv6）：
>
> ```bash
> a.gtld-servers.net. 172800  IN  A   192.5.6.30
> a.gtld-servers.net. 172800  IN  AAAA    2001:503:a83e:0:0:0:2:30
> b.gtld-servers.net. 172800  IN  A   192.33.14.30
> b.gtld-servers.net. 172800  IN  AAAA    2001:503:231d:0:0:0:2:30
> c.gtld-servers.net. 172800  IN  A   192.26.92.30
> c.gtld-servers.net. 172800  IN  AAAA    2001:503:83eb:0:0:0:0:30
> ...
> ```
>
> 理论上，所有域名查询都必须先查询根域名，因为只有根域名才能告诉你，某个顶级域名由哪个运营商、哪台服务器管理。事实上也确实如此，ICANN 维护着[根区文件（Root zone file）](https://www.internic.net/domain/root.zone)，里面记载着顶级域名和对应的记录。
>
> 但由于该文件很少变化，大多数 DNS 服务商都会提供它的缓存，所以根域名的查询事实上不是那么频繁。

## 根区数据库（Root Zone Database）

https://www.iana.org/domains/root/db

> 提供顶级域名的授权信息。
>
> Much of this data is also available via the WHOIS protocol at [whois.iana.org](http://whois.iana.org/).

## 顶级域名（Top-Level Domain (TLD)）

所有顶级域名保存在 [Top-Level Domain List](https://data.iana.org/TLD/tlds-alpha-by-domain.txt)，可查询[根区数据库（Root Zone Database）](https://www.iana.org/domains/root/db)以获得更多信息（如顶级域名运营商信息、Name Servers）。

顶级域名（TLD）分为如下六种类型。其中这两类是最常用的：

* 通用顶级域名（gTLD）
* 国家代码顶级域名（ccTLD）

> As of 2015, IANA distinguishes the following groups of [top-level domains](https://en.wikipedia.org/wiki/Top-level_domain):[\[13\]](https://en.wikipedia.org/wiki/Top-level_domain#cite_note-13)
>
> 1. [Infrastructure top-level domain](https://en.wikipedia.org/wiki/Top-level_domain#Infrastructure_domain) (ARPA): This group consists of one domain, the [Address and Routing Parameter Area](https://en.wikipedia.org/wiki/.arpa). It is managed by IANA on behalf of the [Internet Engineering Task Force](https://en.wikipedia.org/wiki/Internet_Engineering_Task_Force) for various purposes specified in the [Request for Comments](https://en.wikipedia.org/wiki/Request_for_Comments) publications.
>
> 2. [Generic top-level domains](https://en.wikipedia.org/wiki/Generic_top-level_domain) (gTLD): Top-level domains with three or more characters
>
> 3. restricted generic top-level domains (grTLD): These domains are managed under official ICANN accredited registrars.
>
> 4. [Sponsored top-level domains](https://en.wikipedia.org/wiki/Sponsored_top-level_domain) (sTLD): These domains are proposed and sponsored by private agencies or organizations that establish and enforce rules restricting the eligibility to use the TLD. Use is based on community theme concepts; these domains are managed under official ICANN accredited registrars.
>
> 5. [country-code top-level domains](https://en.wikipedia.org/wiki/Country_code_top-level_domain) (ccTLD): Two-letter domains established for [countries](https://en.wikipedia.org/wiki/Country) or [territories](https://en.wikipedia.org/wiki/List_of_dependent_territories). With some historical exceptions, the code for any territory is the same as its two-letter [ISO 3166](https://en.wikipedia.org/wiki/ISO_3166) code.
>    - [Internationalized country code top-level domains](https://en.wikipedia.org/wiki/Internationalized_country_code_top-level_domain) (IDN ccTLD): ccTLDs in non-Latin character sets (e.g., Arabic, Cyrillic, Hebrew, or Chinese).
>
>
> 6. [Test top-level domains](https://en.wikipedia.org/wiki/.test) (tTLD): These domains were installed under [.test](https://en.wikipedia.org/wiki/.test) for testing purposes in the IDN development process; these domains are not present in the root zone.

# 域名解析方式

参考 [RFC 1034: Domain Names - Concepts and Facilities - IETF](https://www.ietf.org/rfc/rfc1034.txt) 规范的 2. INTRODUCTION - 2.3. Assumptions about usage 对于两种域名解析方式的描述：

> In any system that has a distributed database, a particular name server may be presented with a query that can only be answered by some other server.  The two general approaches to dealing with this problem are 
>
> * "recursive", in which the first server pursues the query for the client at another server, 
> * "iterative", in which the server refers the client to another server and lets the client pursue the query.  
>
> Both approaches have advantages and disadvantages, but the iterative approach is preferred for the datagram style of access.  The domain system requires implementation of the iterative approach, but allows the recursive approach as an option.

## 迭代方式（Iterative Approach）

![Iterative Approach](/img/network/application-layer/dns/Example_of_an_iterative_DNS_resolver.svg)

> A DNS resolver that implements the iterative approach mandated by RFC 1034; in this case, the resolver consults three name servers to resolve the [fully qualified domain name](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) "www.wikipedia.org".

## 递归方式（Recursive Approach）

![DNS resolution](/img/network/application-layer/dns/DNS_resolution.png)

# 公共域名服务器

https://en.wikipedia.org/wiki/Public_recursive_name_server

https://public-dns.info/

# DNS 相关命令

## dig

![dig](/img/network/application-layer/dns/dig.png)

`dig` 命令安装：

```bash
$ apt-get install dnsutils
```

`dig` 命令使用：https://downloads.isc.org/isc/bind9/cur/9.17/doc/arm/html/manpages.html#dig-dns-lookup-utility

```bash
$ dig @name_server name record_type
```

`record_type` 参数如下：

![DNS record types](/img/network/application-layer/dns/DNS_record_types.png)

`dig` 命令使用例子：

```bash
# To use a specific DNS server for the query, use the @ option.
$ dig @8.8.8.8 example.com
```

```bash
# By default, dig displays the A record for a domain. To look up a different DNS record, add it to the end of the command. 
# For example, to look up the MX (mail exchanger) record for the example.com domain, type the following command:
$ dig example.com MX
```

```bash
# query for any type of record information
$ dig +noall +answer www.google.com any

www.google.com.		107	IN	A	142.250.66.132
www.google.com.		94	IN	AAAA	2404:6800:4005:802::2004
```

`+[no]trace`

> This option toggles tracing of the delegation path from the root name servers for the name being looked up. Tracing is disabled by default. When tracing is enabled, `dig` makes **iterative queries** to resolve the name being looked up. It follows referrals from the root servers, showing the answer from each server that was used to resolve the lookup.

下例中，使用公共域名服务器，而不是依次使用 [/etc/resolv.conf](https://man7.org/linux/man-pages/man5/resolv.conf.5.html) 里配置的本地域名服务器进行 DNS 迭代查询，结果如下：

```
$ dig @8.8.8.8 +trace www.google.com

; <<>> DiG 9.10.6 <<>> @8.8.8.8 +trace www.google.com
; (1 server found)
;; global options: +cmd
.			24814	IN	NS	m.root-servers.net.
.			24814	IN	NS	b.root-servers.net.
.			24814	IN	NS	c.root-servers.net.
.			24814	IN	NS	d.root-servers.net.
.			24814	IN	NS	e.root-servers.net.
.			24814	IN	NS	f.root-servers.net.
.			24814	IN	NS	g.root-servers.net.
.			24814	IN	NS	h.root-servers.net.
.			24814	IN	NS	a.root-servers.net.
.			24814	IN	NS	i.root-servers.net.
.			24814	IN	NS	j.root-servers.net.
.			24814	IN	NS	k.root-servers.net.
.			24814	IN	NS	l.root-servers.net.
;; Received 525 bytes from 8.8.8.8#53(8.8.8.8) in 157 ms

com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
;; Received 1174 bytes from 202.12.27.33#53(m.root-servers.net) in 48 ms

google.com.		172800	IN	NS	ns2.google.com.
google.com.		172800	IN	NS	ns1.google.com.
google.com.		172800	IN	NS	ns3.google.com.
google.com.		172800	IN	NS	ns4.google.com.
;; Received 840 bytes from 192.48.79.30#53(j.gtld-servers.net) in 196 ms

www.google.com.		300	IN	A	142.250.66.100
;; Received 59 bytes from 216.239.38.10#53(ns4.google.com) in 84 ms
```

Wireshark 网卡抓包截图如下：DNS 协议 基于 UDP 协议，使用 53 端口。

![DNS 协议](/img/network/application-layer/dns/DNS_protocol.png)

## nslookup

nslookup 命令：https://downloads.isc.org/isc/bind9/cur/9.17/doc/arm/html/manpages.html#nslookup-query-internet-name-servers-interactively

```bash
$ nslookup www.google.com

Server:		10.0.20.101
Address:	10.0.20.101#53

Non-authoritative answer:
Name:	www.google.com
Address: 142.250.66.132
```

# 参考

https://www.iana.org/domains/root

[RFC 1034: Domain Names - Concepts and Facilities - IETF](https://www.ietf.org/rfc/rfc1034.txt)

[RFC 1035 - Domain names - implementation and specification](https://datatracker.ietf.org/doc/html/rfc1035)

https://en.wikipedia.org/wiki/Domain_Name_System

https://en.wikipedia.org/wiki/Domain_name

《[根域名的知识 - 阮一峰](https://www.ruanyifeng.com/blog/2018/05/root-domain.html)》

https://wizardzines.com/comics/dig/

https://wizardzines.com/comics/dns-record-types/



《[如果美国封了 DNS，俄罗斯将从网络消失？](https://mp.weixin.qq.com/s/VCzEFqPR9JXAJPUDQv0gVA)》

《[美国能让中国从网络上消失？- 良许 Linux](https://mp.weixin.qq.com/s/if4SfF7gEPH_U6ue_xQHdw)》