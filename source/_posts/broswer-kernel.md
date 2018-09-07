---
title: "浏览器与内核总结"
date: 2015-04-06 11:48:19
updated: 
tags: 前端
description: "工作中需要测试 JavaScript 浏览器兼容性，简单总结如下。"
---

工作中需要测试 JavaScript 浏览器兼容性，简单总结如下。

# 内核介绍

| 内核      | 备注                                       |
| ------- | ---------------------------------------- |
| Trident | IE 内核，Trident 内核长期一家独大，很多国内浏览器使用。        |
| Webkit  | 苹果公司自己的内核，也是苹果的 Safari 浏览器使用的内核。Chrome 也曾使用此内核。 |
| Blink   | 2013年4月3日谷歌称将与苹果 Webkit 分道扬镳，将自主研发 Blink 渲染引擎（即浏览器核心），内置于 Chrome 浏览器之中。 |
| Gecko   | Firefox 使用的内核。                           |
| Presto  | Opera 前内核(已废弃)，现已改用 Webkit 内核。           |

# 浏览器及其内核

| 浏览器                  | IE7  | IE8  | IE9  | IE10 | Webkit | Gecko | 备注                                 |
| -------------------- | ---- | ---- | ---- | ---- | ------ | ----- | ---------------------------------- |
| Internet Explorer 7  | √    |      |      |      |        |       |                                    |
| Internet Explorer 8  |      | √    |      |      |        |       |                                    |
| Internet Explorer 9  |      |      | √    |      |        |       |                                    |
| Internet Explorer 10 |      |      |      | √    |        |       |                                    |
| Google Chrome        |      |      |      |      | √      |       |                                    |
| Mozilla Firefox      |      |      |      |      |        | √     |                                    |
| Apple Safari         |      |      |      |      | √      |       |                                    |
| Opera                |      |      |      |      | √      |       | Opera 放弃自家 Presto 内核转投 WebKit      |
| 360极速浏览器             | √    |      |      | √    | √      |       |                                    |
| 360安全浏览器             |      |      |      | √    | √      |       |                                    |
| 傲游云浏览器               |      |      |      | √    | √      |       | 通过菜单，切换浏览内核：兼容->极速 操作              |
| QQ浏览器                | √    | √    | √    | √    | √      |       | 通过兼容性视图切到 IE7，通过启用扩展组件调用 Webkit 内核 |
| 搜狐高速浏览器              | √    |      |      |      | √      |       | 在地址栏点击切换“高速”或“兼容”模式                |

# 参考

* 《[国内各IE内核浏览器所调用的IE版本](http://www.feelcss.com/domestic-ie-core-browser-called-ie-version.html)》
* 《[浏览器检测工具](http://castic.cyscc.org/help/browserdetecter/)》