---
title: Java 虚拟机系列（二）运行时数据区总结
date: 2019-09-01 11:20:46
updated:
tags: [Java, JVM]
typora-root-url: ..
---

# 背景

最近为了做春节大型活动，研究了下性能压测和 JVM 调优，先来看一张 JVM 监控图（硬件：4 核 8G）。

![JVM 监控](/img/java/jvm/jvm_monitor.png)

6 小时的吞吐量为：(21600s - Young GC 35s + Old GC 0s) / 21600s = 99.8%，总吞吐量还是不错的（吞吐量就是 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值，即 `吞吐量 = 运行用户代码时间 / (运行用户代码时间 + GC 时间)`。如果虚拟机总共运行了 100 分钟，其中 GC 总耗时 1 分钟，那么吞吐量就是 99%。）。同时单次 Young GC 的平均耗时仅为 35s / 639 = 55 ms，停顿时间较短。

如果还需要进一步优化，思路如下：

* 合理调整 Old Gen 与 Young Gen 大小比例，以减少 Young GC 次数，但单次 GC 耗时可能会相应延长，具体需测试。
* 更换垃圾收集器，并对垃圾收集器参数进行调优。

下面介绍一些基础知识。

# 运行时数据区

手绘的运行时数据区如下：

![jvm](/img/java/jvm/runtime_data_areas.png)

JVM 定义了在程序执行期间使用的各种运行时数据区：

* 其中一些数据区是在 JVM 启动时创建、仅在 JVM 退出时才被销毁。
* 另外一些数据区是随每个线程创建及销毁。

# `java` Command

**java** [*options*] *classname* [*args*]

**java** [*options*] **-jar** *filename* [*args*]

* *options*: Command-line options separated by spaces.
* *classname*: The name of the class to be launched.
* *filename*: The name of the Java Archive (JAR) file to be called. Used only with the `-jar` option.
* *args*: The arguments passed to the `main()` method separated by spaces.

`java` 命令用于启动 Java 应用程序。它通过启动 JRE，加载指定类并调用其 `main()` 方法来实现启动。`main()` 方法声明如下：

```java
public static void main(String[] args)
```

> `java` 命令支持以下几类选项：
>
> - [标准选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABDJJFI)，JVM 的所有实现所支持的最常用选项。
> - [非标准选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABHDABI)，特定于 Java HotSpot VM 的通用选项。
> - [高级运行时选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABCBGHF)，用于控制 Java HotSpot VM 的运行时行为。
> - [高级 JIT 编译器选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABDDFII)，用于控制 Java HotSpot VM 执行动态实时（JIT）编译。
> - [高级可维修性选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABFJDIC)，提供了收集系统信息和执行调试的能力。
> - [高级垃圾收集选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABFAFAE)，用于控制 Java HotSpot VM 如何执行垃圾回收（GC）。
>
> 说明：
>
> - 所有 JVM 实现都需要保证支持*标准选项*。标准选项用于执行常见操作，例如检查 JRE 版本、设置类路径、启用详细输出等。
> - *非标准选项*是针对 Java HotSpot VM 的通用选项，因此不能保证所有 JVM 实现都能支持，并且随时可能改变。非标组选项以 `-X` 开头。
> - *高级选项*不建议随意使用。这些是开发人员用于调整 Java HotSpot VM 特定区域的选项。这些区域通常具有特定的系统要求，并且可能需要对系统配置参数的访问权限。这些选项也不能保证所有 JVM 实现都能支持，并且随时可能改变。高级选项以 `-XX` 开头。
>
> 想跟踪最新版本中被弃用或删除的选项，参考[已废弃与已移除的选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABDCEGG)（JDK 8）。
>

> 布尔类型的选项无需参数，格式如下：
>
> * `-XX:+`*OptionName* 用于启用 默认情况下禁用的功能；
> * `-XX:-`*OptionName* 用于禁用 默认情况下启用的功能。
>
> 对于需要参数的选项，每个选项的确切语法有所差异：
>
> * 参数可以用空格、冒号（`:`）或等号（`=`）与选项名分开，或者参数可以直接跟在选项后面，具体参考文档。

> 如果需要指定字节大小，可以使用以下几种格式：
>
> * no suffix
> * `k` or `K` for kilobytes (KB)
> * `m` or `M` for megabytes (MB)
> * `g` or `G` for gigabytes (GB)
>
> 例如：
>
> - 大小为 8 GB，参数可以设为 `8g`, `8192m`, `8388608k`, `8589934592`。
> - 大小为 1.5 GB，参数不能设为 `1.5g`，可以设为 `1536m`。
> - 如果需要指定百分比，使用 0 到 1 之间的数字（例如， `0.25` for 25%）。

## JVM Stack

`-Xss`*size*

> Sets the thread stack size (in bytes). Append the letter `k` or `K` to indicate KB, `m` or `M` to indicate MB, `g` or `G` to indicate GB. The default value depends on the platform:
>
> - Linux/ARM (32-bit): `320 KB`
> - Linux/i386 (32-bit): `320 KB`
> - Linux/x64 (64-bit): `1024 KB`
> - OS X (64-bit): `1024 KB`
> - Oracle Solaris/i386 (32-bit): `320 KB`
> - Oracle Solaris/x64 (64-bit): `1024 KB`
>
> The following examples set the thread stack size to `1024 KB` in different units:
>
> ```
> -Xss1m
> -Xss1024k
> -Xss1048576
> ```
>
> This option is equivalent to `-XX:ThreadStackSize`.
>

## Heap

| 参数                                                        | 描述                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| `-Xms`、`-XX:InitialHeapSize`<br/>`-Xmx`、`-XX:MaxHeapSize` | 设置 Heap 堆区的初始值和最大值，Server 端 JVM 建议将 `-Xms` 和 `-Xmx` 设为相同值。 |

| 参数   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| `-Xmn` | 设置 Heap 堆内 Young Generation，而 Old Generation 等于：堆区减去 `-Xmn`。<br/>设置 `-Xmn` 等同于设置了相同的 Young Generation 初始值 `-XX:NewSize` 和最大值 `-XX:MaxNewSize`。 |

| 参数                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| `-XX:NewRatio`       | Sets the ratio between **young and old generation** sizes. By default, this option is set to `2` (Young Gen can get up to 1/3 (`Y=H/(R+1)`) of the Heap). |
| ` -XX:SurvivorRatio` | Sets the ratio between **eden and survivor space** sizes. By default, this option is set to `8` (S0/S1 can get up to 1/10 (`S=Y/(R+2)`) of Young Gen). |

### `-Xms`、`-Xmx`

`-Xms`*size*

> Sets the initial size (in bytes) of the heap. This value must be a multiple of 1024 and greater than 1 MB. Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes.
>
> The following examples show how to set the size of allocated memory to 6 MB using various units:
>
> ```
> -Xms6291456
> -Xms6144k
> -Xms6m
> ```
>
> If you do not set this option, then the initial size will be set as the sum of the sizes allocated for the old generation and the young generation.
>
> The `-Xms` option is equivalent to `-XX:InitialHeapSize`.
>

`-Xmx`*size*

> Specifies the maximum size (in bytes) of the memory allocation pool in bytes. This value must be a multiple of 1024 and greater than 2 MB. Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes. The default value is chosen at runtime based on system configuration. 
>
> > For server deployments, `-Xms` and `-Xmx` are often set to the same value. See the section "Ergonomics" in *Java SE HotSpot Virtual Machine Garbage Collection Tuning Guide* at http://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/index.html.
>
> The following examples show how to set the maximum allowed size of allocated memory to 80 MB using various units:
>
> ```
> -Xmx83886080
> -Xmx81920k
> -Xmx80m
> ```
>
> The `-Xmx` option is equivalent to `-XX:MaxHeapSize`.
>

### `-Xmn`

`-Xmn`*size*

> Sets the initial and maximum size (in bytes) of the heap for the **young generation** (nursery).
>
> > The young generation region of the heap is used for new objects. GC is performed in this region more often than in other regions. 
> >
> > * If the size is too small, then a lot of minor garbage collections will be performed. 
> > * If the size is too large, then only full garbage collections will be performed, which can take a long time to complete. 
> >
> > ⚠️ **Oracle recommends that you keep the size for the young generation between a half and a quarter of the overall heap size.**
> 
> Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes. The following examples show how to set the initial and maximum size of young generation to 256 MB using various units:
>
> ```
> -Xmn256m
> -Xmn262144k
> -Xmn268435456
> ```
>
> Instead of the `-Xmn` option to set both the initial and maximum size of the heap for the young generation, you can use `-XX:NewSize` to set the initial size and `-XX:MaxNewSize` to set the maximum size.
>

### `-XX:NewRatio`

`-XX:NewRatio`=*ratio*

Sets the ratio between **young and old generation** sizes. By default, this option is set to `2`.

> The `NewRatio` is the ratio of old generation to young generation (e.g. value 2 means max size of old will be twice the max size of young, i.e. young can get up to 1/3 of the heap).
>
> ```
> Y=H/(R+1)
> ```
> 

设置 Young Generation 和 Old Generation 的比值，例如该值默认为 2，则表示 Young Generation 和 Old Generation 比值为1:2。

### ` -XX:SurvivorRatio`

`-XX:SurvivorRatio`=*ratio*

Sets the ratio between **eden and survivor space** sizes. By default, this option is set to `8`.

> The following formula can be used to calculate the initial size of survivor space (S) based on the size of the young generation (Y), and the initial survivor space ratio (R):
>
> ```
> S=Y/(R+2)
> ```
>
> The 2 in the equation denotes two survivor spaces. The larger the value specified as the initial survivor space ratio, the smaller the initial survivor space size.
>
> By default, the initial survivor space ratio is set to 8. If the default value for the young generation space size is used (2 MB), the initial size of the survivor space will be 0.2 MB.

## Method Area

### PermGen

JDK 7 及以下版本：

| 参数              | 描述          |
| ----------------- | ------------- |
| `-XX:PermSize`    | Perm 的初始值 |
| `-XX:MaxPermSize` | Perm 的最大值 |

JVM 的永久代(PermGen)主要用于存放 Class 的 meta-data，Class 在被 Loader 加载时就会被放到 PermGen space，GC 在主程序运行期间不会对该区进行清理，默认是 64M 大小，当程序需要加载的对象比较多时，超过 64M 就会报这部分内存溢出了，需要加大内存分配。 

### Metaspace

JDK 8 及以上版本，永久代（PermGen）的概念被废弃掉了，参考 [JEP 122: Remove the Permanent Generation](http://openjdk.java.net/jeps/122)：

> The proposed implementation will **allocate class meta-data in native memory** and **move interned Strings and class static variables to the Java heap**. 
>
> Hotspot will explicitly allocate and free the native memory for the class meta-data. Allocation of new class meta-data would be **limited by the amount of available native memory** rather than fixed by the value of `-XX:MaxPermSize`, whether the default or specified on the command line.

取而代之的是一个称为 Metaspace 的存储空间。Metaspace 使用的是本地内存，而不是堆内存，也就是说在默认情况下 Metaspace 的大小只与本地内存大小有关。可以通过以下的几个参数对 Metaspace 进行控制： 

| 参数                     | 描述               |
| ------------------------ | ------------------ |
| ` -XX:MetaspaceSize `    | Metaspace 的初始值 |
| ` -XX:MaxMetaspaceSize ` | Metaspace 的最大值 |

## Direct Memory

`-XX:MaxDirectMemorySize`=*size*

> Sets the maximum total size (in bytes) of the New I/O (the `java.nio` package) direct-buffer allocations. Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes. By default, the size is set to 0, meaning that the JVM chooses the size for NIO direct-buffer allocations automatically.
>
> The following examples illustrate how to set the NIO size to 1024 KB in different units:
>
> ```
> -XX:MaxDirectMemorySize=1m
> -XX:MaxDirectMemorySize=1024k
> -XX:MaxDirectMemorySize=1048576
> ```

# 异常

## java.lang.StackOverflowError

`java.lang.StackOverflowError`：线程栈溢出，要么是方法调用层次过多（比如存在无限递归调用）：

![StackOverflow](/img/java/jvm/SOF.png)

要么是线程栈太小，可调整 `-Xss` 参数增加线程栈大小。

## java.lang.OutOfMemoryError: Java heap space

`java.lang.OutOfMemoryError: Java heap space`：这种是堆内存不够，一个原因是真不够，另一个原因是程序中有死循环，例如：

![OutOfMemory](/img/java/jvm/OOM.png)

如果是堆内存不足，可调整 `-Xms`、`-Xmx`，或者新老生代的比例。

## java.lang.OutOfMemoryError: PermGen space

`java.lang.OutOfMemoryError: PermGen space`：这种是P区内存不够，可调整：`-XX:PermSize`、`-XX:MaxPermSize`。

# 参考

《[Java 虚拟机规范（Java SE 8 版 - 中文版）](https://item.jd.com/11703581.html)》

《[Java 虚拟机规范（Java SE 8 版 - 英文版）](https://docs.oracle.com/javase/specs/jvms/se8/html/index.html)》

《[JEP 122: Remove the Permanent Generation - Release on JDK 8](http://openjdk.java.net/jeps/122)》

《[Command Line Options - JDK 8 HotSpot VM](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)》