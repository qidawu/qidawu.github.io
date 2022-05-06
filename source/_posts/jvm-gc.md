---
title: Java 虚拟机系列（三）垃圾收集器总结
date: 2019-09-10 23:22:23
updated:
tags: [Java, JVM]
typora-root-url: ..
---

本文总结下垃圾收集涉及的一些重点：

![gc_summary](/img/java/jvm/gc_summary.png)

基于分代收集算法的垃圾收集器组合，总结如下图，常用于 JDK 8 及之前的版本：

![generational_collection](/img/java/jvm/gc_combination.png)

# 参考

《深入理解 Java 虚拟机》