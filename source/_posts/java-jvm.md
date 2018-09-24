---
title: JVM 内存分区总结
date: 2017-01-29 11:20:46
updated:
tags: Java
---

最近为了做春节大型活动，研究了下性能压测和 JVM 调优，先来看一张 JVM 监控图。

![JVM 监控](/img/java/jvm_monitor.png)

可以看出，内存分配不太合理，Old 区和 Metaspace 可以合理减小、增加新生代空间以减少 Young GC 次数，提升压测性能。

下面介绍一些基础知识。

# JVM 内存区域

![JVM 内存区域](/img/java/jvm.png)

# JVM 堆区

各区描述如下：

![JVM 堆区](/img/java/hotspot_heap_structure.png)

![JVM 堆区](/img/java/jvm_space.jpg)

堆内存 = 

- 新生代（Eden 区 + 两个 Survivor 区（From、To）） + 
- 老年代（Old 区） + 
- 永久代（Perm 区）

Heap = 

- Young Generation (Eden Space + From Space + To Space) + 
- Old Generation (Old Space) + 
- Perm Generation (Perm Space)

如果是 JDK 8+ 还有些调整，后面介绍。



Tomcat 参数可以调整 JVM 各区：

## 堆

| 参数           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| `-Xms`、`-Xmx` | 设置 Heap 堆（E+S0+S1+O）的初始值和最大值，Server 端 JVM 建议将 `-Xms` 和 `-Xmx` 设为相同值； |

## 新生代、老年代

| 参数   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| `-Xmn` | 设置 Heap 堆内新生代（E+S0+S1），而老年代（O）等于：`-Xmx` 减去 `-Xmn`。<br/>设置 `-Xmn` 等同于设置了相同的新生代初始值 `-XX:NewSize` 和最大值 `-XX:MaxNewSize`。 |

| 参数                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| `-XX:NewRatio`       | 设置新生代和老年代的比值，例如该值为 3，则表示新生代和老年代比值为1:3。 |
| ` -XX:SurvivorRatio` | 设置新生代中 E 区和 S 区的比例， 即 -XX:SurvivorRatio=eden/s0=eden/s1。 |

## 永久代

JDK 7 及以下：

| 参数              | 描述          |
| ----------------- | ------------- |
| `-XX:PermSize`    | Perm 的初始值 |
| `-XX:MaxPermSize` | Perm 的最大值 |

JVM 的 Perm 区主要用于存放 Class 和 Meta 信息的，Class 在被 Loader 加载时就会被放到 PermGen space，GC 在主程序运行期间不会对该区进行清理，默认是 64M 大小，当程序需要加载的对象比较多时，超过 64M 就会报这部分内存溢出了，需要加大内存分配，一般 128m 足够。 

## 元空间

JDK 8 开始，永久代(PermGen)的概念被废弃掉了，取而代之的是一个称为 Metaspace 的存储空间。Metaspace 使用的是本地内存，而不是堆内存，也就是说在默认情况下 Metaspace 的大小只与本地内存大小有关。可以通过以下的几个参数对 Metaspace 进行控制： 

| 参数                     | 描述               |
| ------------------------ | ------------------ |
| ` -XX:MetaspaceSize `    | Metaspace 的初始值 |
| ` -XX:MaxMetaspaceSize ` | Metaspace 的最大值 |

# 内存溢出

## 情况一

`java.lang.OutOfMemoryError: Java heap space`：这种是堆内存不够，一个原因是真不够，另一个原因是程序中有死循环，例如：

![OutOfMemory](/img/java/OOM.png)

如果是堆内存不足，可调整 `-Xms`、`-Xmx`，或者新老生代的比例。

## 情况二

`java.lang.OutOfMemoryError: PermGen space`：这种是P区内存不够，可调整：`-XX:PermSize`、`-XX:MaxPermSize`

## 情况三

`java.lang.StackOverflowError`：线程栈溢出，要么是方法调用层次过多（比如存在无限递归调用）：

![StackOverflow](/img/java/SOF.png)

要么是线程栈太小，可调整 `-Xss` 参数增加线程栈大小。