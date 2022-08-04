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

# 字符串的处理

## 字符串拼接

字符串拼接的三种方式，区别如下：

|                           | since       | immutable ? | thread-safe ?  |
| ------------------------- | ----------- | ----------- | -------------- |
| `java.lang.String`        | Java SE 1.0 | immutable   | no thread-safe |
| `java.lang.StringBuffer`  | Java SE 1.0 | mutable     | thread-safe    |
| `java.lang.StringBuilder` | Java SE 1.5 | mutable     | no thread-safe |

字符串拼接的字节码分析，参考：[《On Java 8》第十八章 字符串：`+` 的重载与 `StringBuilder` 对比](https://zyb0408.github.io/gitbooks/onjava8/docs/book/18-Strings.html#%E9%87%8D%E8%BD%BD%E5%92%8CStringBuilder)

除了上述方式之外， Java SE 8 还提供了新的工具类 `java.util.StringJoiner`，它是`String.join` 和 `java.util.stream.Collectors#joining(...)` 的底层实现。结合 Stream API 使用如下：

```java
// [a,b,c,d,e,f,g]
Arrays.asList("a", "b", "c", "d", "e", "f", "g").stream()
                .collect(Collectors.joining(",", "[", "]"));
```

## 字符串格式化

[`java.util.Formatter`](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html) —— C 语言 `printf` 风格的字符串格式化解释器。

![Formatter](/img/java/string/Formatter.png)

用法：

Returns a formatted string using the specified `format` string and `args`.

```java
String s1 = new Formatter().format(format, args).toString();
// or simplied
String s2 = String.format(format, args);
```

Writes a formatted string to this **output stream** using the specified `format` string and `args`.

```java
System.out.printf(format, args);
System.out.format(format, args);
```

参考：

https://linux.die.net/man/1/printf

https://linux.die.net/man/3/printf

https://www.baeldung.com/linux/printf-echo

```bash
$ printf "%s %s" hello world | hexyl
┌────────┬─────────────────────────┬─────────────────────────┬────────┬────────┐
│00000000│ 68 65 6c 6c 6f 20 77 6f ┊ 72 6c 64                │hello wo┊rld     │
└────────┴─────────────────────────┴─────────────────────────┴────────┴────────┘
```



https://www.baeldung.com/java-printstream-printf

https://blog.csdn.net/quinnnorris/article/details/54614446

## 字符串转义

方式一：在线工具：https://www.freeformatter.com/string-escaper.html

> HTML Escape
> Escapes or unescapes an HTML file removing traces of offending characters that could be wrongfully interpreted as markup.
>
> > FEATURES
> > Escapes all reserverd characters with their corresponding HTML entities (', ", &, <, >)
> > Escapes ISO 8859-1 symbols and characters that have corresponding HTML entities
>
> JSON Escape
> Escapes or unescapes a JSON string removing traces of offending characters that could prevent parsing.
>
> XML Escape
> Escapes or unescapes an XML file removing traces of offending characters that could be wrongfully interpreted as markup.
>
> CSV Escape
> Escapes or unescapes a CSV string removing traces of offending characters that could prevent parsing.
>
> JavaScript Escape
> Escapes or unescapes a JavaScript string removing traces of offending characters that could prevent interpretation.
>
> Java and .Net Escape
> Escapes or unescapes a Java or .Net string removing traces of offending characters that could prevent compiling.
>
> SQL Escape
> Escapes or unescapes a SQL string removing traces of offending characters that could prevent execution.

方式二：代码处理

Apache Commons Lang 提供了字符串转义和反转义工具类 `org.apache.commons.lang3.StringEscapeUtils`，用于 Java、JavaScript、JSON、HTML、XML、CSV 等字符串：

![StringEscapeUtils](/img/java/string/StringEscapeUtils.png)

例如：

```java
// &lt;span&gt;hello world&lt;/span&gt;
StringEscapeUtils.escapeHtml4("<span>hello world</span>");

// {hello: "world"}
StringEscapeUtils.unescapeJson("{hello: \"world\"}");
```

例如，有些 HTML 字符实体在 PDF 是不支持的，需要先转义：

```java
// flying saucer
ITextRenderer renderer = new ITextRenderer();

String escapedHtml = StringEscapeUtils.escapeHtml4(srcHtml);
renderer.setDocumentFromString(escapedHtml);
```

参考：

https://en.wikipedia.org/wiki/Escape_sequence

https://en.wikipedia.org/wiki/Escape_sequences_in_C

## 字符串操作

Apache Commons Lang 为字符串操作提供了 `org.apache.commons.lang3.StringUtils` 工具类，使用方式参考[这里](/2017/12/25/apache-commons-lang/)。

![StringUtils](/img/java/commons/commons-lang/StringUtils.png)

# 参考

《[On Java 8 - 第十八章 字符串](https://lingcoder.github.io/OnJava8/#/book/18-Strings)》

《[Java String 对象，你真的了解了吗？](https://cloud.tencent.com/developer/article/1511298)》

《[几张图轻松理解 `String.intern()`](https://blog.csdn.net/tyyking/article/details/82496901)》

《[再议String-字符串常量池与String.intern()](https://mp.weixin.qq.com/s/vkP-JXMs12i1QBVdnI4KJQ)》

《[字符串常量池深入解析](https://blog.csdn.net/weixin_40304387/article/details/81071816)》

《[JVM 常量池、运行时常量池、字符串常量池](https://www.cnblogs.com/natian-ws/p/10749164.html)》



《[Java8 对字符串连接的改进](https://segmentfault.com/a/1190000007835105)》

《[HTML 字符实体](http://www.w3school.com.cn/html/html_entities.asp)》