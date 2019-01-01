---
title: 浏览器同源策略及跨域解决方案
date: 2017-06-13 14:01:08
updated:
tags: 前端
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

# 跨域资源共享

由于 XMLHttpRequest 受同源策略的约束，不能跨域访问资源，因此 W3C 委员会制定了 XMLHttpRequest 跨域访问标准 [Cross-Origin Resource Sharing](http://en.wikipedia.org/wiki/Cross-Origin_Resource_Sharing)，通过目标域返回的 HTTP 头（Access-Control-Allow-Origin）来授权是否允许跨域访问。由于 HTTP 头对于 JavaScript 来说，一般是无法控制的，所以认为这个方案可以安全施行。

# 参考

https://en.wikipedia.org/wiki/Same-origin_policy

https://en.wikipedia.org/wiki/Cross-origin_resource_sharing

http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html

http://www.ruanyifeng.com/blog/2016/04/cors.html

https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy

http://blog.csdn.net/wang379275614/article/details/53333054