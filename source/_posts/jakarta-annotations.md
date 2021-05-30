---
title: Jakarta EE 系列（二）Annotations 通用注解规范总结
date: 2020-10-03 23:32:08
updated:
tags: [Java]
typora-root-url: ..
---

> Jakarta Annotations defines a collection of annotations representing common semantic concepts that enable a declarative style of programming that applies across a variety of Java technologies.

通过在 Java 平台中添加 JSR 175（Java 编程语言的元数据工具），我们设想各种技术将使用注解来实现声明式编程风格。如果这些技术各自为共同概念独立定义自己的注解，那将是不幸的。在 Jakarta EE 和 Java SE 组件技术中保持一致性很有价值，但在 Jakarta EE 和 Java SE 之间实现一致性也很有价值。

本规范的目的是定义一小组通用注解，这些注解可在其它规范中使用。希望这将有助于避免在不同 Jakarta EE 规范中定义的注解之间不必要的冗余或重复。这将允许我们将通用注解集中在一个地方，让技术引用此规范，而不是在多个规范中指定它们。这样，所有技术都可以使用相同版本的注解，并且跨平台使用的注解将保持一致。

这些通用注解详见如下：

* [javax.annotation](https://jakarta.ee/specifications/annotations/1.3/apidocs/)
* [jakarta.annotation](https://jakarta.ee/specifications/annotations/2.0/apidocs/)

