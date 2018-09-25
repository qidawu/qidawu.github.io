---
title: JVM 垃圾收集总结
date: 2017-02-05 23:22:23
updated:
tags: Java
---

# 判断对象是否可回收

![GC Root Set](/img/java/gc/gc_roots.png)

可做 gc roots 的对象：

* 栈中引用对象
* 方法区静态类属性引用对象
* 方法区常量引用对象

# 垃圾收集算法

## mark-sweep

![mark-sweep](/img/java/gc/mark_sweep.png)

## mark-compact

![mark-compact](/img/java/gc/mark_compact.png)

## coping

![coping](/img/java/gc/coping.png)

# 垃圾收集器

## 垃圾收集器组合

![垃圾收集器组合](/img/java/gc/gc_combine.png)

## serial/serial old

![serial/serial old](/img/java/gc/serial&serial_old.png)

## parnew/serial old

![parnew/serial old](/img/java/gc/parnew&serial_old.png)

## cms

![cms](/img/java/gc/cms.png)