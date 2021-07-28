---
title: 浏览器同源策略及跨域解决方案
date: 2017-10-13 14:01:08
updated: 2021-07-27 14:01:08
tags: [安全, 前端]
---

# 同源策略

要了解什么是同源策略，先来看下如果没有同源策略，我们将遇到什么安全问题：

>假设用户正在访问银行网站并且没有退出登录。然后，用户打开新 Tab 转到另一个恶意网站，其中有一些恶意 JavaScript 代码在后台运行并请求来自银行网站的数据。由于用户仍然在银行网站登录，恶意代码可以模拟用户在银行网站上做任何事情。这是因为浏览器可以向同域的银行网站发送和接收会话cookie。

同源策略（Same Origin Policy）是一种约定，是浏览器最核心也最基本的安全功能。如果缺少了同源策略，则浏览器的正常功能可能都会受到影响，可以说 Web 是构建在同源策略的基础之上的，浏览器只是针对同源策略的一种实现。

同源策略限制了来自不同源站的“document”或脚本，对当前“document”读取或设置某些属性。受同源策略限制的元素：

* Cookie
* DOM
* XMLHttpRequest
* 第三方插件
  * Adobe Flash
  * Java Applet
  * Microsoft Silverlight
  * Google Gears
  * ……

一个域名的组成：

| http:// |      |      |      |      |
| ------- | ---- | ---- | ---- | ---- |
|         |      |      |      |      |

## Chrome 增强的网站隔离功能 [桌面版 / Android]

> 因为同源政策（Same Origin Policy）的存在，A 网站一般无法访问 B 网站存储在设备中的网站数据。不过因为安全漏洞的存在，部分恶意网站偶尔也可以绕过同源政策、获取这些文件并对其他网站进行攻击。
>
> 所以除了第一时间对各种浏览器安全漏洞进行修补，Chrome 也在早些时候引入了网站隔离功能：通过在独立进程中加载网站页面，确保网站与网站之间的数据安全性。在 Chrome 92 稳定版的桌面端，这个功能从浏览器标签页延伸到了浏览器扩展，不同扩展插件也将在独立进程中进行加载，同时几乎不会影响大部分扩展的现有功能。
>
> 出于性能考虑，网站隔离功能在 Android 版 Chrome 中一直都没有像桌面端那样全盘开启，仅在一些需要用户手动输入登录信息的网站中启用，并且仅支持拥有 2GB 及以上运行内存的设备。
>
> 本次 Chrome 92 则通过对 OAuth 2.0 协议和 Cross-Origin-Opener-Policy (COOP) 策略的额外支持扩展了网站隔离功能在移动端上的可用性和兼容性，换句话说，移动版 Chrome 现在能够识别并保护更多类型的网站了（比如采用第三方登录的那种）。
>
> 不过增强网站隔离功能在 Android 端的硬件限制依然为 2GB RAM，Google 同时也表示，如果你的设备可用内存足够，也可以通过 chrome://flags#enable-site-per-process 这一功能标签来手动开启全盘网站隔离。

2021-07-27 Updated

# 跨域资源共享

由于 XMLHttpRequest 受同源策略的约束，不能跨域访问资源，因此 W3C 委员会制定了 XMLHttpRequest 跨域访问标准 [Cross-Origin Resource Sharing](http://en.wikipedia.org/wiki/Cross-Origin_Resource_Sharing)，通过目标域返回的 HTTP 头（Access-Control-Allow-Origin）来授权是否允许跨域访问。由于 HTTP 头对于 JavaScript 来说，一般是无法控制的，所以认为这个方案可以安全施行。

# 参考

https://en.wikipedia.org/wiki/Same-origin_policy

https://en.wikipedia.org/wiki/Cross-origin_resource_sharing

http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html

http://www.ruanyifeng.com/blog/2016/04/cors.html

https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy

http://blog.csdn.net/wang379275614/article/details/53333054