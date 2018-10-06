---
title: H5 存储的四种方案介绍
date: 2017-01-07 16:21:15
updated:
tags: 前端
---

![h5 存储](/img/javascript/h5_storage.png)

其中 Cookie 的 `domain` 属性比较特殊，存在一些读写限制：可读写本身或上一级 `domain` 的 cookie，但无法读写同级或下一级 `domain` 的 cookie。

|           | z.com | y.z.com |
| --------- | ----- | ------- |
| z.com     | √     | √       |
| y.z.com   | ×     | √       |
| x.z.com   | ×     | ×       |
| x.y.z.com | ×     | ×       |

