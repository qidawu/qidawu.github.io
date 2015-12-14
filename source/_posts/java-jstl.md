title: "JSP 标准标签库（JSTL）总结"
date: 2015-05-02 22:05:51
updated: 
tags: Java
---

JSP 标准标签库（JSP Standard Tag Library）是一个 JSP 标签集合，它封装了 JSP 应用的通用核心功能。

它的出现，是因为人们开始注重软件的分层设计，不希望在 JSP 页面中出现 JAVA 逻辑代码。同时也由于自定义标签的开发难度较大、不利于技术的标准化，因此产生了 JSTL。

JSTL 和 EL 的结合，基本可以让页面再无 `<% %>` 代码。

JSTL 标准标签库可分为五类：

# 核心标签库

共 14 个，从功能上可以分为 4 类。引用方法：

```
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```

## 表达式控制

|标签|描述|
|---|---|
|`<c:out>`|用于显示数据，就像 `<%= %>`，区别在于 `<c:out>` 标签可以直接通过 `.` 操作符来访问属性|
|`<c:set>`|用于保存数据|
|`<c:remove>`|用于删除数据|
|`<c:catch>`|用来处理产生错误的异常状况，并且将错误信息储存起来|

## 流程控制

|标签|描述|
|---|---|
|`<c:if>`|与我们在一般程序中用的 `if` 一样|
|`<c:choose>`|本身只当做 `<c:when>` 和 `<c:otherwise>` 的父标签|
|`<c:when>`|`<c:choose>` 的子标签，用来判断条件是否成立|
|`<c:otherwise>`|`<c:choose>` 的子标签，接在 `<c:when>` 标签后，当 `<c:when>` 标签判断为 `false` 时被执行|

## 循环

|标签|描述|
|---|---|
|`<c:forEach>`|基础迭代标签，接受多种集合类型|
|`<c:forTokens>`|根据指定的**分隔符**来分隔内容并迭代输出|

## URL 操作

|标签|描述|
|---|---|
|`<c:import>`|检索一个绝对或相对 URL，然后将其内容暴露给页面|
|`<c:url>`|使用可选的查询参数来创造一个 URL|
|`<c:redirect>`|重定向至一个新的 URL|
|`<c:param>`|用来给包含或重定向的页面传递参数|

# 格式化标签库

用于格式化并输出文本、日期、时间、数字，这里只介绍最最最常用的两个标签。引用方法：

```
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
```

## 格式化数字

|标签|描述|
|---|---|
|`<fmt:formatNumber>`|使用指定的格式或精度格式化数字|

## 格式化日期

|标签|描述|
|---|---|
|`<fmt:formatDate>`|使用指定的风格或模式格式化日期和时间|

# SQL 标签库

不常用。引用方法：

```
<%@ taglib prefix="sql" uri="http://java.sun.com/jsp/jstl/sql" %>
```

# XML 标签库

不常用。引用方法：

```
<%@ taglib prefix="x" uri="http://java.sun.com/jsp/jstl/xml" %>
```

# 函数标签库

大部分都是通用的字符串处理函数，用于配合 **EL 表达式**使用。引用方法：

```
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
```

|标签|描述|
|---|---|
|`fn:contains()`|测试输入的字符串是否包含指定的子串|
|`fn:containsIgnoreCase()`|测试输入的字符串是否包含指定的子串，大小写不敏感|
|`fn:endsWith()`|测试输入的字符串是否以指定的后缀结尾|
|`fn:escapeXml()`|跳过可以作为XML标记的字符|
|`fn:indexOf()`|返回指定字符串在输入字符串中出现的位置|
|`fn:join()`|将数组中的元素合成一个字符串然后输出|
|`fn:length()`|返回字符串长度|
|`fn:replace()`|将输入字符串中指定的位置替换为指定的字符串然后返回|
|`fn:split()`|将字符串用指定的分隔符分隔然后组成一个子字符串数组并返回|
|`fn:startsWith()`|测试输入字符串是否以指定的前缀开始|
|`fn:substring()`|返回字符串的子集|
|`fn:substringAfter()`|返回字符串在指定子串之后的子集|
|`fn:substringBefore()`|返回字符串在指定子串之前的子集|
|`fn:toLowerCase()`|将字符串中的字符转为小写|
|`fn:toUpperCase()`|将字符串中的字符转为大写|
|`fn:trim()`|移除首位的空白符|

# 参考

《[JSP 标准标签库（JSTL）](http://www.runoob.com/jsp/jsp-jstl.html)》