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

![jvm](/img/java/jvm/runtime_data_areas.jpg)

JVM 定义了在程序执行期间使用的各种运行时数据区：

* 其中一些数据区是在 JVM 启动时创建、仅在 JVM 退出时才被销毁。
* 另外一些数据区是随每个线程创建及销毁。

## PC Register

JVM 可以一次支持多个线程执行。每个 JVM 线程都有专属的 `pc`（程序计数器 program counter）寄存器。在任何时候，每个 JVM 线程都在执行某个方法的代码，即该线程的当前方法（参考 [栈帧（Frames）](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)一节）。

`pc` 寄存器的值可以为两种：

* 如果当前执行的是非 `native` 方法，值为当前正在执行的 JVM 指令的地址（`returnAddress`）。
* 如果是 `native` 方法，值为 undefined。

JVM 的 `pc` 寄存器长度足以保存 `returnAddress` 或特定平台的本地指针。



The `returnAddress` Type and Values :

> The `returnAddress` type is used by the Java Virtual Machine's *jsr*, *ret*, and *jsr_w* instructions ([§*jsr*](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.jsr), [§*ret*](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.ret), [§*jsr_w*](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.jsr_w)). The values of the `returnAddress` type are pointers to the opcodes of Java Virtual Machine instructions. Unlike the numeric primitive types, the `returnAddress` type does not correspond to any Java programming language type and cannot be modified by the running program.

## JVM Stacks

每个 JVM 线程都有一个私有的 *JVM Stack* 栈区，与该线程同时创建。JVM Stack 存储栈帧（参考 [栈帧（Frames）](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)一节）。JVM Stack 类似于常规编程语言（例如 C 语言）：它保存局部变量和部分结果，并在方法调用和返回中起作用。由于 JVM Stack 的操作只有出栈和入栈，因此栈帧可能堆积。JVM Stack 的内存空间不必连续。

规范允许 JVM Stack 要么是固定大小（通过 `-Xss` 指定大小）、要么是根据计算的需要进行动态扩容和缩容。如果 JVM Stack 的大小固定，则在创建每个 JVM Stack 时可以独立选择其大小。

以下异常情况与 JVM Stack 相关：

* 如果线程所需空间大于分配的 JVM Stack 空间，则 JVM 抛出 `StackOverflowError`。
* 如果 JVM Stack 能够动态扩展，并尝试扩展，但是内存不足，则 JVM 抛出  `OutOfMemoryError`。

### Frames

栈帧用于：

* 存储数据和部分结果
* 作为方法的返回值
* 调度异常
* 执行动态链接

每次调用方法时都会创建一个栈帧。当方法调用完毕，无论是正常还是异常结束（例如抛出了未捕获的异常），栈帧都会销毁。栈帧由 JVM 栈区创建。每个栈帧都有它自己的局部变量数组（Local Variables Array）、操作数栈（Operand Stacks）、以及对当前类当前方法的运行时常量池的引用。

局部变量数组和操作数栈的大小在编译时确定，并与栈帧关联的方法代码（参考 [The `Code` Attribute](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.3)）一起提供。因此，栈帧数据结构的大小仅取决于 JVM 的实现，并且可以在方法调用时分配用于这些结构的内存。

在给定线程的任何时候，只有一个在执行方法中的栈帧处于活动状态。该活动栈帧称为*当前帧（current frame）*，该方法称为*当前方法（current method）*，定义当前方法的类称为*当前类（current class）*。局部变量和操作数栈上的操作引用*当前帧*。

如果当前方法调用了另一个方法或者该方法执行完毕，则该方法所处的帧不再是*当前帧*。调用方法时，将创建新的栈帧，并在控制权转移到新方法时变为*当前帧*。当方法返回时，*当前帧*将其方法调用的结果（如有）传递回前一帧并被丢弃，然后前一帧变回*当前帧*。

注意，由线程创建的栈帧仅线程自身可见，无法被其它线程所引用。

## Heap

JVM 具有一个在所有 JVM 线程之间共享的堆区（*Heap*），用于分配所有类实例和数组所需的内存。

堆区在虚拟机启动时创建。堆中的对象由垃圾收集器（*garbage collector*）进行回收。对象永远不会显式释放。JVM 不假定任何类型的垃圾收集器，而由实现者根据系统要求自行选择实现。

堆的大小可以是固定的，也可以根据计算的需要进行扩容，如果不需要更大空间的堆，可以进行缩容。堆区的内存空间不必连续。

JVM 实现可以为用户提供堆的初始值配置。并且，如果堆可以动态扩容和缩容，还需提供堆的最大、最小值配置。

以下异常情况与堆相关：

* 如果所需的堆空间大于能够分配的堆空间，则 JVM 抛出  `OutOfMemoryError`。

## Method Area

JVM 具有一个在所有 JVM 线程之间共享的方法区（*Method Area*）。方法区类似于常规编程语言的编译代码的存储区域。它存储每个类的结构，例如运行时常量池、字段（field）及方法（method）的数据、以及方法（methods）和构造方法（constructors）的代码，包括用于类及其实例初始化和接口初始化的特殊方法（参考 [Special Methods](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.9)）。 

方法区在虚拟机启动时创建。尽管方法区在逻辑上是堆区的一部分，但是 JVM 实现可以选择不进行垃圾回收或压缩。JVM 规范没有规定方法区的位置或用于管理已编译代码的策略。

方法区可以是固定大小的，或者根据计算的需要进行扩容，如果无需更大空间的方法区，可以进行缩容。方法区的内存空间不必连续。

JVM 实现可以为用户提供方法区的初始值配置。在方法区大小可变的情况下，可以提供最大、最小值配置。

以下异常情况与方法区相关：

* 如果方法区的内存空间无法满足分配请求，则 JVM 抛出  `OutOfMemoryError`。

## Run-Time Constant Pool

运行时常量池是每个类或每个接口的 `class` 文件中 `constant_pool` 表（参考 [The Constant Pool](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4)）的运行时表示。它包含多种常量，范围从编译时已知的数值型的字面值（*numeric literals*）到必须在运行时解析的方法和字段引用。运行时常量池的功能类似于常规编程语言的符号表，尽管它包含的数据范围比典型的符号表要大。

每个运行时常量池都由方法区分配。当 JVM 创建类或接口时（参考 [Creation and Loading Class](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.3)），将构造该类或接口的运行时常量池。

以下异常情况与方法区相关：

* 创建类或接口时，如果运行时常量池的构造所需的内存超过 JVM 的方法区中可用的内存，则 JVM 抛出  `OutOfMemoryError`。

## Native Method Stacks

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

`java` 命令支持以下几类选项：

- [标准选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABDJJFI)，JVM 的所有实现所支持的最常用选项。
- [非标准选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABHDABI)，特定于 Java HotSpot VM 的通用选项。
- [高级运行时选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABCBGHF)，用于控制 Java HotSpot VM 的运行时行为。
- [高级 JIT 编译器选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABDDFII)，用于控制 Java HotSpot VM 执行动态实时（JIT）编译。
- [高级可维修性选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABFJDIC)，提供了收集系统信息和执行调试的能力。
- [高级垃圾收集选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABFAFAE)，用于控制 Java HotSpot VM 如何执行垃圾回收（GC）。

所有 JVM 实现都需要保证支持*标准选项*。标准选项用于执行常见操作，例如检查 JRE 版本、设置类路径、启用详细输出等。

*非标准选项*是针对 Java HotSpot VM 的通用选项，因此不能保证所有 JVM 实现都能支持，并且随时可能改变。非标组选项以 `-X` 开头。

*高级选项*不建议随意使用。这些是开发人员用于调整 Java HotSpot VM 特定区域的选项。这些区域通常具有特定的系统要求，并且可能需要对系统配置参数的访问权限。这些选项也不能保证所有 JVM 实现都能支持，并且随时可能改变。高级选项以 `-XX` 开头。

想跟踪最新版本中被弃用或删除的选项，参考[已废弃与已移除的选项](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABDCEGG)（JDK 8）。

布尔类型的选项用于启用默认情况下禁用的功能，或者禁用默认情况下启用的功能。此类选项无需参数，格式如下：

* `-XX:+`*OptionName* 用于启用；
* `-XX:-`*OptionName* 用于禁用。

对于需要参数的选项，每个选项的确切语法有所差异：参数可以用空格、冒号（`:`）或等号（`=`）与选项名分开，或者参数可以直接跟在选项后面，具体参考文档。

如果需要指定字节大小，可以使用以下几种格式：

* no suffix
* `k` or `K` for kilobytes (KB)
* `m` or `M` for megabytes (MB)
* `g` or `G` for gigabytes (GB)

例如，大小为 8 GB，参数可以设为 `8g`, `8192m`, `8388608k`, `8589934592`。如果需要指定百分比，使用 0 到 1 之间的数字（例如， `0.25` for 25%）。

## JVM Stack

`-Xss`*size*

> Sets the thread stack size (in bytes). Append the letter `k` or `K` to indicate KB, `m` or `M` to indicate MB, `g` or `G` to indicate GB. The default value depends on the platform:
>
> - Linux/ARM (32-bit): 320 KB
> - Linux/i386 (32-bit): 320 KB
> - Linux/x64 (64-bit): 1024 KB
> - OS X (64-bit): 1024 KB
> - Oracle Solaris/i386 (32-bit): 320 KB
> - Oracle Solaris/x64 (64-bit): 1024 KB
>
> The following examples set the thread stack size to 1024 KB in different units:
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
| `-XX:NewRatio`       | Sets the ratio between **young and old generation** sizes. By default, this option is set to 2. |
| ` -XX:SurvivorRatio` | Sets the ratio between **eden and survivor space** sizes. By default, this option is set to 8. |

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

> Sets the initial and maximum size (in bytes) of the heap for the young generation (nursery). Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes.
>
> > The young generation region of the heap is used for new objects. GC is performed in this region more often than in other regions. 
> >
> > * If the size is too small, then a lot of minor garbage collections will be performed. 
> > * If the size is too large, then only full garbage collections will be performed, which can take a long time to complete. 
> >
> > **Oracle recommends that you keep the size for the young generation between a half and a quarter of the overall heap size.**
>
> The following examples show how to set the initial and maximum size of young generation to 256 MB using various units:
>
> ```
> -Xmn256m
> -Xmn262144k
> -Xmn268435456
> ```
>
> ⚠️ Instead of the `-Xmn` option to set both the initial and maximum size of the heap for the young generation, you can use `-XX:NewSize` to set the initial size and `-XX:MaxNewSize` to set the maximum size.
>

### `-XX:NewRatio`

`-XX:NewRatio`=*ratio*

Sets the ratio between **young and old generation** sizes. By default, this option is set to 2.

> The `NewRatio` is the ratio of old generation to young generation (e.g. value 2 means max size of old will be twice the max size of young, i.e. young can get up to 1/3 of the heap).
>
> ```
> Y=H/(R+1)
> ```
> 
> 设置 Young Generation 和 Old Generation 的比值，例如该值默认为 2，则表示 Young Generation 和 Old Generation 比值为1:2。

### ` -XX:SurvivorRatio`

`-XX:SurvivorRatio`=*ratio*

Sets the ratio between **eden and survivor space** sizes. By default, this option is set to 8.

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

JDK 7 以前：

| 参数              | 描述          |
| ----------------- | ------------- |
| `-XX:PermSize`    | Perm 的初始值 |
| `-XX:MaxPermSize` | Perm 的最大值 |

JVM 的永久代(PermGen)主要用于存放 Class 的 meta-data，Class 在被 Loader 加载时就会被放到 PermGen space，GC 在主程序运行期间不会对该区进行清理，默认是 64M 大小，当程序需要加载的对象比较多时，超过 64M 就会报这部分内存溢出了，需要加大内存分配。 

### Metaspace

JDK 8 及以后，永久代(PermGen)的概念被废弃掉了，参考 [JEP 122: Remove the Permanent Generation](http://openjdk.java.net/jeps/122)：

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

Sets the maximum total size (in bytes) of the New I/O (the `java.nio` package) direct-buffer allocations. Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes. By default, the size is set to 0, meaning that the JVM chooses the size for NIO direct-buffer allocations automatically.

The following examples illustrate how to set the NIO size to 1024 KB in different units:

```
-XX:MaxDirectMemorySize=1m
-XX:MaxDirectMemorySize=1024k
-XX:MaxDirectMemorySize=1048576
```

# 内存溢出

## 情况一

`java.lang.OutOfMemoryError: Java heap space`：这种是堆内存不够，一个原因是真不够，另一个原因是程序中有死循环，例如：

![OutOfMemory](/img/java/jvm/OOM.png)

如果是堆内存不足，可调整 `-Xms`、`-Xmx`，或者新老生代的比例。

## 情况二

`java.lang.OutOfMemoryError: PermGen space`：这种是P区内存不够，可调整：`-XX:PermSize`、`-XX:MaxPermSize`。

## 情况三

`java.lang.StackOverflowError`：线程栈溢出，要么是方法调用层次过多（比如存在无限递归调用）：

![StackOverflow](/img/java/jvm/SOF.png)

要么是线程栈太小，可调整 `-Xss` 参数增加线程栈大小。

# 参考

《[Java 虚拟机规范（Java SE 8 版 - 中文版）](https://item.jd.com/11703581.html)》

《[Java 虚拟机规范（Java SE 8 版 - 英文版）](https://docs.oracle.com/javase/specs/jvms/se8/html/index.html)》

《[JEP 122: Remove the Permanent Generation - Release on JDK 8](http://openjdk.java.net/jeps/122)》

《[Command Line Options - JDK 8 HotSpot VM](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)》