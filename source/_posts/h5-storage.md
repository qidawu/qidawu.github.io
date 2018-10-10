---
title: H5 存储的四种方案介绍
date: 2017-01-07 16:21:15
updated:
tags: 前端
---

![h5 存储](/img/javascript/h5_storage.png)

Cookie 指令在 HTTP 头的形式如下：

- HTTP 请求头 `Cookie` 指令：

  ```
  Cookie: code=23365f1409b; __auth=eda2ebe49a4a91d3546435c3
  ```

- HTTP 响应头 `Set-Cookie` 指令：

  ```
  Set-Cookie: code=23365f1409b; Expires=Thu, 31-Dec-2016 07:23:59 GMT; Domain=x.y.z.com; Path=/ws; Secure; HttpOnly
  ```

其中 Cookie 的 `domain` 属性比较特殊，存在一些读写限制：可读写本身或上一级 `domain` 的 cookie，但无法读写同级或下一级 `domain` 的 cookie。

|           | z.com | y.z.com |
| --------- | ----- | ------- |
| z.com     | √     | √       |
| y.z.com   | ×     | √       |
| x.z.com   | ×     | ×       |
| x.y.z.com | ×     | ×       |

如果不设置 Cookie 的 `Expires` 或者 `Max-Age` 属性，其默认值是 Session，也就是关闭浏览器后该 Cookie 就消失了。

# 参考

https://www.ibm.com/developerworks/cn/java/books/javaweb_xlb/10/index.html

http://www.cnblogs.com/xiaowei0705/archive/2011/04/19/2021372.html

