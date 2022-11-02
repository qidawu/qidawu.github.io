---
title: Java 调优篇（一）JVM 规范总结
date: 2018-08-31 22:20:46
updated:
tags: Java
typora-root-url: ..
---

JVM 是 Java 平台的基石。

JVM 对 Java 编程语言一无所知，仅知道特定的二进制格式，即 `class` 文件格式（class file format）。一个 `class` 文件包含 JVM 指令（或字节码 bytecodes）、一张符号表、以及其它辅助信息。

为了安全起见，JVM 对 `class` 文件中的代码施加了严格的语法和结构约束。然而，任何能够转为有效 `class` 文件的语言都能由 JVM 托管。受此特性吸引（通用、独立于机器的平台），其它语言的实现者可以将 JVM 用作其语言的交付工具。

# JVM 结构

要正确实现 JVM，仅需要能够读取 `class` 文件格式并正确执行其中指定的操作。非 JVM 规范的实现细节，例如，运行时数据区的内存布局、使用的 GC 算法、任何 JVM 指令的内部优化（例如编译为机器码）则由规范实现者自行决定。

## class 文件格式

JVM 要执行的已编译代码（Compiled code）使用独立于硬件和操作系统的二进制格式即字节码（bytecodes）表示，通常存储在称为 `class` 文件格式的文件中。该 `class` 文件格式精确定义了类或接口，其中包括各种细节。

## 数据类型

就像 Java 编程语言一样，JVM 可对两种类型进行操作：*原始类型（primitive types）* 和*引用类型（reference types）*。可用于变量存储、参数传递、方法返回、并对其进行操作。

JVM 希望几乎所有类型检查都在运行时之前由编译器完成，而不必由 JVM 本身完成。

# 运行时数据区

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

### Run-Time Constant Pool

运行时常量池是每个类或每个接口的 `class` 文件中 `constant_pool` 表（参考 [The Constant Pool](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4)）的运行时表示。它包含多种常量，范围从编译时已知的数值型的字面值（*numeric literals*）到必须在运行时解析的方法和字段引用。运行时常量池的功能类似于常规编程语言的符号表，尽管它包含的数据范围比典型的符号表要大。

每个运行时常量池都由方法区分配。当 JVM 创建类或接口时（参考 [Creation and Loading Class](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.3)），将构造该类或接口的运行时常量池。

以下异常情况与方法区相关：

* 创建类或接口时，如果运行时常量池的构造所需的内存超过 JVM 的方法区中可用的内存，则 JVM 抛出  `OutOfMemoryError`。
