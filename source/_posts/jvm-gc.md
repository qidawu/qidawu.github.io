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

# GC 选项配置

https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

* *DefNew* – single-threaded mark-copy stop-the-world garbage collector and is what is used to clean the Young generation（单线程 (single-threaded), 采用标记-复制 (mark-copy) 算法的，使整个 JVM 暂停运行 (stop-the-world) 的新生代 (Young generation) 垃圾收集器 (garbage collector)）

| Advanced Garbage Collection Options | GC Configuration                 | Description                                                  |                                                              |
| ----------------------------------- | -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-XX:+UseSerialGC`                  | Serial + Serial Old              | Enables the use of the serial garbage collector. This is generally the best choice for small and simple applications that do not require any special functionality from garbage collection. <br/>By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM. |                                                              |
| ~~`-XX:+UseParNewGC`~~              | ~~ParNew + SerialOld~~           | Enables the use of parallel threads for collection in the **young generation**. <br/>By default, this option is disabled. It is automatically enabled when you set the `-XX:+UseConcMarkSweepGC` option. Using the `-XX:+UseParNewGC` option without the `-XX:+UseConcMarkSweepGC` option was deprecated in JDK 8. | Java 8 [JEP 173: Retire Some Rarely-Used GC Combinations](https://openjdk.org/jeps/173)<br/>Java 9 [JEP 214: Remove GC Combinations Deprecated in JDK 8](https://openjdk.org/jeps/214) |
| ~~`-XX:+UseConcMarkSweepGC`~~       | ~~ParNew + CMS~~                 | Enables the use of the CMS garbage collector for the **old generation**. Oracle recommends that you use the CMS garbage collector when application latency requirements cannot be met by the throughput (`-XX:+UseParallelGC`) garbage collector. The G1 garbage collector (`-XX:+UseG1GC`) is another alternative.<br/>By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM. <br/>When this option is enabled, the `-XX:+UseParNewGC` option is automatically set and you should not disable it, because the following combination of options has been deprecated in JDK 8: `-XX:+UseConcMarkSweepGC -XX:-UseParNewGC`. (Serial + CMS) | Java 9 [JEP 291: Deprecate the Concurrent Mark Sweep (CMS) Garbage Collector](https://openjdk.org/jeps/291)<br/>Java 14 [JEP 363: Remove the Concurrent Mark Sweep (CMS) Garbage Collector](https://openjdk.org/jeps/363) |
| `-XX:+UseParallelGC`                | Parallel Scavenge + Parallel Old | Enables the use of the parallel scavenge garbage collector (also known as the throughput collector) to improve the performance of your application by leveraging multiple processors.<br/>By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM. If it is enabled, then the `-XX:+UseParallelOldGC` option is automatically enabled, unless you explicitly disable it. | Java 14 [JEP 366: Deprecate the ParallelScavenge + SerialOld GC Combination](https://openjdk.org/jeps/366) |
| `-XX:+UseParallelOldGC`             | Parallel Scavenge + Parallel Old | Enables the use of the parallel garbage collector for full GCs. <br/>By default, this option is disabled. Enabling it automatically enables the `-XX:+UseParallelGC` option. |                                                              |
| `-XX:+UseG1GC`                      |                                  | Enables the use of the garbage-first (G1) garbage collector. It is a server-style garbage collector, targeted for multiprocessor machines with a large amount of RAM. It meets GC pause time goals with high probability, while maintaining good throughput. The G1 collector is recommended for applications requiring large heaps (sizes of around 6 GB or larger) with limited GC latency requirements (stable and predictable pause time below 0.5 seconds).<br/>By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM. |                                                              |
| `-XX:UseZGC`                        |                                  |                                                              |                                                              |

## G1

[Garbage First Garbage Collector Tuning - Oracle](https://www.oracle.com/technical-resources/articles/java/g1gc.html)

## GC 日志

Java 9 [JEP 158: Unified JVM Logging](https://openjdk.org/jeps/158)

Java 9 [JEP 271: Unified GC Logging](https://openjdk.org/jeps/271)

# 参考

[HotSpot Virtual Machine Garbage Collection Tuning Guide - Java 17](https://docs.oracle.com/en/java/javase/17/gctuning/index.html)

> [Available Collectors](https://docs.oracle.com/en/java/javase/17/gctuning/available-collectors.html)
>
> - Serial Collector
> - Parallel Collector
> - Garbage-First (G1) Garbage Collector
> - The Z Garbage Collector

https://www.baeldung.com/java-verbose-gc

《深入理解 Java 虚拟机》

[从 Java 8 升级到 Java 17 踩坑全过程](https://mp.weixin.qq.com/s/d0U_d7D9RzEfn1u5STBNHg)