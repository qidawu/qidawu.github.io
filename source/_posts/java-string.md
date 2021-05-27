---
title: Java 常用类型系列（三）字符串类型总结
date: 2018-04-04 22:56:42
updated:
tags: Java
typora-root-url: ..
---

# 继承结构

![methods_of_CharSequence](/img/java/string/methods_of_CharSequence.png)

![Appendable](/img/java/string/Appendable.png)

![CharSequence](/img/java/string/CharSequence.png)

# 字符串的实现

![history_of_String](/img/java/string/history_of_String.jpg)

# 字符串的创建方式

## 通过字符串常量创建

## 通过构造方法创建

## `intern()` 方法

# 字符串的特性

三种常用字符串类的区别如下：

|                           | since       | immutable ? | thread-safe ?  |
| ------------------------- | ----------- | ----------- | -------------- |
| `java.lang.String`        | Java SE 1.0 | immutable   | no thread-safe |
| `java.lang.StringBuffer`  | Java SE 1.0 | mutable     | thread-safe    |
| `java.lang.StringBuilder` | Java SE 1.5 | mutable     | no thread-safe |

## 字符串拼接-字节码分析

[+ 的重载与 StringBuilder](https://lingcoder.github.io/OnJava8/#/book/18-Strings?id=-%e7%9a%84%e9%87%8d%e8%bd%bd%e4%b8%8e-stringbuilder)

# 工具类

## java.util.Formatter

C 语言 `printf` 风格的字符串格式化解释器。用法参考[这里](https://blog.csdn.net/quinnnorris/article/details/54614446)。

https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html

![Formatter](/img/java/string/Formatter.png)

## java.util.StringJoiner

对于字符串拼接，除了使用 `StringBuilder` 之外， Java SE 8 还提供了新的 API `java.util.StringJoiner`，它是`String.join` 和 `java.util.stream.Collectors#joining(...)` 的底层实现。结合 Stream API 使用如下：

```java
// [a,b,c,d,e,f,g]
Arrays.asList("a", "b", "c", "d", "e", "f", "g").stream()
                .collect(Collectors.joining(",", "[", "]"));
```

## org.apache.commons.lang3.StringUtils

Apache Commons Lang 为字符串操作提供了 `StringUtils` 工具类，使用方式参考[这里](/2017/12/25/apache-commons-lang/)。

![StringUtils](/img/java/commons/commons-lang/StringUtils.png)

## org.apache.commons.lang3.StringEscapeUtils

Apache Commons Lang 提供了字符串转义和反转义工具类 `StringEscapeUtils`，用于 Java、JavaScript、JSON、HTML、XML、CSV 等字符串：

![StringEscapeUtils](/img/java/string/StringEscapeUtils.png)

例如：

```java
// &lt;span&gt;hello world&lt;/span&gt;
StringEscapeUtils.escapeHtml4("<span>hello world</span>");

// {hello: "world"}
StringEscapeUtils.unescapeJson("{hello: \"world\"}");
```

# 参考

《[On Java 8 - 第十八章 字符串](https://lingcoder.github.io/OnJava8/#/book/18-Strings)》

《[Java String 对象，你真的了解了吗？](https://cloud.tencent.com/developer/article/1511298)》

《[Java8 对字符串连接的改进](https://segmentfault.com/a/1190000007835105)》

《[HTML 字符实体](http://www.w3school.com.cn/html/html_entities.asp)》