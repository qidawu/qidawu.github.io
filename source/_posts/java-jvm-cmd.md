---
title: JVM 监控常用命令
tags: Java
date: 2017-02-16 20:10:12
updated:
---


# jps

`jps` (JVM Process Status Tool) 命令的功能与 `ps` 类似，用于列出正在运行的 JVM 进程状态。

常用参数：

- `-q` 只输出 LVMID
- `-v` 输出虚拟机进程启动时 JVM 参数

# jstat

`jstat` (JVM Statistics Monitoring Tool) 命令用于监控 JVM 的性能统计信息。

命令格式：

```bash
$ jstat Options vmid [interval[s|ms] [count]]
```

常用参数：

- Options，常用选项：
  - `-gccapacity` 查看 GC 情况和 JVM 各区的**容量**（字节）
  - `-gc` 查看 GC 情况和 JVM 各区的**容量**和**使用量**（字节）
  - `-gcutil` 查看 GC 情况和 JVM 各区的**使用率**（%）
  - `[-h<lines>]` 每 N 行显示一次标题
- vmid，VM 的进程号，即当前运行的 Java 进程号
- interval，间隔时间，单位为秒或者毫秒
- count，打印次数，如果缺省则打印无数次

示例展示：

此示例连接到 lvmid 21891，并以 250 毫秒的间隔获取 7 个样本，并显示由 `-gcutil` 选项指定的输出：

```bash
$ jstat -gcutil -h6 21891 250 7

  S0     S1     E      O      P     YGC    YGCT    FGC    FGCT     GCT

 12.44   0.00  27.20   9.49  96.70    78    0.176     5    0.495    0.672

 12.44   0.00  62.16   9.49  96.70    78    0.176     5    0.495    0.672

 12.44   0.00  83.97   9.49  96.70    78    0.176     5    0.495    0.672

  0.00   7.74   0.00   9.51  96.70    79    0.177     5    0.495    0.673

  0.00   7.74  23.37   9.51  96.70    79    0.177     5    0.495    0.673

  0.00   7.74  43.82   9.51  96.70    79    0.177     5    0.495    0.673

  S0     S1     E      O      P     YGC    YGCT    FGC    FGCT     GCT

  0.00   7.74  58.11   9.51  96.71    79    0.177     5    0.495    0.673
```

该示例结果显示，在第 3 和第 4 个样本之间发生了 Young GC。 gc 耗时 0.001 秒，并将对象从 Eden 区（E）提升到 Old 区（O），导致 Old 区的使用率从 9.49％ 增加到 9.51％。 在 gc 之前，Survivor 区（S0）的使用率为 12.44％，但在 gc 之后仅使用了 7.74％（S1）。

 `-gcutil` 选项每列说明：

![JVM 堆内存布局](/img/java/jvm_space.jpg)

堆内存（Heap） = 年轻代 + 年老代 + 永久代
年轻代 = Eden 区 + 两个 Survivor 区（From 和 To）

| 列名   | 描述                                  |
| ---- | ----------------------------------- |
| S0   | Heap 上的 Survivor space 0 区（单位**%**） |
| S1   | Heap 上的 Survivor space 1 区（单位**%**） |
| E    | Heap 上的 Eden space 区（单位**%**）       |
| O    | Heap 上的 Old space 区（单位**%**）        |
| P    | Perm space 区（单位**%**）               |

| 列名   | 描述                              |
| ---- | ------------------------------- |
| YGC  | 从应用程序启动到采样时发生 Young GC 的次数      |
| YGCT | 从应用程序启动到采样时 Young GC 所用的时间（单位秒） |
| FGC  | 从应用程序启动到采样时发生 Full GC 的次数       |
| FGCT | 从应用程序启动到采样时 Full GC 所用的时间（单位秒）  |
| GCT  | 从应用程序启动到采样时用于垃圾回收的总时间（单位秒）      |

Tomcat 参数调整 JVM 各区：

| 参数              | 描述                                       |
| --------------- | ---------------------------------------- |
| -Xms            | Heap 初始值（S0+S1+E+O），Server 端 JVM 建议将 -Xms 和 -Xmx 设为相同值； |
| -Xmx            | Heap 最大值；                                |
| -Xmn            | Heap 最小值；                                |
| -XX:NewSize     | Heap 新生代（S0+S1+E）的初始值，新生代与老年代的大小比例默认为 2:1； |
| -XX:MaxNewSize  | Heap 新生代的最大值；                            |
| -XX:PermSize    | Perm 初始值；                                |
| -XX:MaxPermSize | Perm 最大值；                                |

# 参考

《[JVM性能调优监控工具jps、jstack、jmap、jhat、jstat、hprof使用详解](https://my.oschina.net/feichexia/blog/196575)》