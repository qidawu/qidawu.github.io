title: Javascript 简介
date: 2016-02-10 16:57:00
updated:
tags: JavaScript
---

一个完整的 JavaScript 实现由下列三个不同的部分组成：

```
+--------------------------+
|                          |
|        JavaScript        |
|                          |
| +----------+ +---+ +---+ |
| |ECMAScript| |DOM| |BOM| |
| +----------+ +---+ +---+ |
|                          |
+--------------------------+
```

下面分别介绍这些部分：

# 核心（ECMAScript）

以网景的 Netscape Navigator 内置的 JavaScript 1.1 为蓝本，由 ECMA-262 定义的 ECMAScript 是一种与 Web 浏览器没有依赖关系的脚本语言标准，它由下列基础部分组成：

* 语法（Syntax）
* 类型（Types）
* 语句（Statements）
* 关键字（Keywords）
* 保留字（Reserved words）
* 操作符（Operators）
* 对象（Objects）

ECMA-262 定义的只是这门语言的基础，而在此基础之上，**宿主环境（host environment）** 可以构建更完善的脚本语言。

## 宿主环境

例如我们常见的 Web 浏览器就是 ECMAScript 实现中可能的宿主环境之一。宿主环境不仅提供基本的 ECMAScript 实现，同时也会提供该语言的扩展，以便语言与环境之间对接交互。而这些扩展——如 DOM，则利用 ECMAScript 的核心类型（Types）和语法（Syntax）提供更多更具体的功能，以便实现针对环境的操作。其它宿主环境还包括：

* Node.js（一种服务端 JavaScript 平台）
* Adobe Flash

## 版本

ECMA-262 目前已经发布了六个大版本的 ECMAScript：

|版本|发布时间|描述|
|---|---|---|
|1|1997 年 6 月|以网景的 Netscape Navigator 内置的 JavaScript 1.1 为蓝本制定，但删除了所有针对浏览器的代码，并支持 Unicode 标准（从而支持多语言开发）。|
|2|1998 年 6 月|基本没有修改。|
|3|1999 年 12 月|标准的第一次大修改，涉及：新增的正则表达式，更好的字符串处理，新的控制语句，try / catch 异常处理的支持，更严格的错误定义，数值格式化输出和其它增强功能。该版标志着 ECMAScript 成为了一门真正的编程语言。|
|~~4~~|~~已废弃~~|该版对 ECMAScript 进行了大刀阔斧的修改，但由于复杂的语言政治分歧而被废弃了。|
|5|2009 年 12 月|澄清了第三版规范许多模糊之处，并增加了一些新功能，如：原生 JSON 对象、继承的方法和高级属性定义，以及“严格模式（strict mode）”。**是目前浏览器兼容性最好、最主流的版本。**|
|5.1|2011 年 6 月|基本没有修改。|
|6|2015 年 6 月|标准的又一次大修改，被称为 ECMAScript 2015。它为编写日益复杂的应用程序增加了大量重要的新语法，包括：类（classes）和模块（modules）、新的迭代器（iterators）和 for/of 循环（loops）、Python 风格的生成器（generators）和生成器表达式、arrow functions, binary data, typed arrays, collections (maps, sets and weak maps), promises, number and math enhancements, reflection, and proxies ...<br/>更多特性详见[这里](http://es6-features.org/#Constants)。|
|7|制定中||

## 兼容

各个浏览器对 ECMAScript 5 的兼容性可查看 [这里](http://caniuse.com/#feat=es5) 。

# 文档对象模型（DOM）

文档对象模型（DOM，Document Object Model）是针对 XML 但经过扩展用于 HTML 的 API。借助 DOM 提供的 API，开发人员可以轻松自如地删除、添加、替换或修改任何节点，获得控制页面内容和结构的主动权。

# 浏览器对象模型（BOM）

浏览器对象模型（BOM，Browser Object Model）是一组浏览器提供的自定义扩展 API，可以控制浏览器显示的页面以外的部分，例如：

* 弹出新浏览器窗口的功能；
* 移动、缩放和关闭浏览器窗口的功能；
* 提供浏览器详细信息的 navigator 对象；
* 提供浏览器所加载页面的详细信息的 location 对象；
* 提供用户显示器分辨率详细信息的 screen 对象；
* 对 cookies 的支持；
* 像 XMLHttpRequest 和 IE 的 ActiveXObject 这样的自定义对象。

常用的 BOM API 如下：

```
window
  |
  +--> document
  |
  +--> location
  |
  +--> navigator
  |
  +--> screen
  |
  +--> history
  |
  +--> ...
```

由于没有 BOM 标准可以遵循，因此每个浏览器都有自己的实现。虽然也存在一些事实标准，例如要有 window 对象和 navigator 对象等，但每个浏览器都会为这两个对象乃至其它对象定义自己的属性和方法。如今 HTML 5 致力于把很多 BOM 功能纳入正式规范，BOM 实现的细节有望朝着兼容性越来越高的方向发展。

# 参考

* 《JavaScript 高级程序设计》
* 《[ECMAScript - Wikipedia](https://en.wikipedia.org/wiki/ECMAScript)》
* 《[Document Object Model - Wikipedia](https://en.wikipedia.org/wiki/Document_Object_Model)》
* 《[Browser Object Model - Wikipedia](https://en.wikipedia.org/wiki/Browser_Object_Model)》