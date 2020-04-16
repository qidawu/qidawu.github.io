---
title: Java 虚拟机系列（四）性能监控、故障处理工具总结
date: 2019-09-16 20:10:12
updated:
tags: Java
typora-root-url: ..
---

命令行工具：

| 命令     | 全称                                                     | 作用                                                         | 备注                          |
| -------- | -------------------------------------------------------- | ------------------------------------------------------------ | ----------------------------- |
| `jps`    | 虚拟机进程状况工具（JVM Process Status Tool），          | 显示正在运行的所有 HotSpot VM 进程。                         |                               |
| `jstat`  | 虚拟机统计信息监控工具（JVM Statistics Monitoring Tool） | 用于监视本地或远程 HotSpot VM 各方面的运行数据，例如类加载/卸载、运行时数据区、GC、JIT。 |                               |
| `jinfo`  | Java 配置信息工具（Configuration Info for Java）         | 实时显示或修改虚拟机配置信息。例如 `jinfo -flag MetaspaceSize VMID` | 在 JDK 9 中已集成到 `JHSDB`   |
| `jmap`   | Memory Map for Java                                      | 用于实时生成虚拟机的堆内存转储快照（heap dump/hprof 文件），或查看堆内存信息。其它转储方法：<br/>`-XX:+HeapDumpOnOutOfMemoryError`<br/>`-XX:+HeapDumpOnCtrlBreak` | 在 JDK 9 中已集成到 `JHSDB`   |
| `jhat`   | JVM Heap Dump Browser                                    | 用于分析 heap dump 文件，它会建立一个 HTTP/HTML 服务器，让用户可以在浏览器上查看分析结果。分析结果默认以包为单位进行分组显示，分析内存泄漏问题主要会使用到其中的 Heap Histogram（与 `jmap -histo` 功能一样）与 OQL 页签功能，前者可以找到内存中总容量最大的对象，后者是标准的对象查询语言，使用类似 SQL 的语法堆内存中的对象进行查询统计。 | 在 JDK 9 中已被  `JHSDB` 替代 |
| `jstack` | Stack Trace for Java                                     | 显示虚拟机当前时刻的线程快照（thread dump/javacore 文件）。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成堆栈快照的目的通常是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间挂起等，都是导致线程长时间停顿的常见原因。 | 在 JDK 9 中已集成到 `JHSDB`   |

可视化工具：

| 命令        | 名称                  | 基于                      | 作用                                                         | 备注             |
| ----------- | --------------------- | ------------------------- | ------------------------------------------------------------ | ---------------- |
| `jconsole`  | Java Console          | JMX                       | 一款基于 JMX (Java Management Extensions) 的可视化监视、管理工具。它的主要功能是通过 JMX  的 MBean (Managed Bean) 对系统进行信息收集和参数动态调整。 | JDK 5 起免费提供 |
| `jvisualvm` | Java VisualVM         | 无需特殊 Agent            | 拥有丰富的插件扩展。目前已经从 Oracle JDK 中分离出来，成为一个独立发展的开源项目：http://visualvm.github.io/ | JDK 6 起免费提供 |
| `jmc`       | Java Mission Control  | Java Flight Recorder, JFR | 曾经是 BEA 公司的图形化诊断工具，随着 BEA 公司被 Oracle 收购而融合进 Oracle JDK。从 JDK 11 开始已被移除出 JDK。2018 年开源并交付给 Open JDK 组织管理。需要与 HotSpot 内部的 JFR 配合才能工作。 | JDK 7 起付费提供 |
| `jhsdb`     | Java HotSpot Debugger | Serviceability Agent      | 一个基于 Serviceability Agent 的 HotSpot 进程调试器。        | JDK 9 起免费提供 |

# 命令行工具

## jps

`jps` 命令的功能与 `ps` 类似，用于列出正在运行的 JVM 进程状态。

常用参数：

- `-q` 只输出 LVMID，省略主类的名称。
- `-l` 输出主类的全名，如果进程执行的是 JAR 包，则输出 JAR 路径。
- `-m` 输出虚拟机进程启动时传递给主类 `main()` 函数的参数。
- `-v` 输出虚拟机进程启动时的 JVM 参数。

## jstat

`jstat` 命令用于监视当前 JVM 的各种运行状态信息。在用户体验上也许不如 JMC、VisualVM 等可视化监控工具以图表形式展示那样直观，但在实际生产环境中不一定可以使用 GUI 图形界面，因此在没有 GUI、只提供命令行界面的服务器上，仍是**运行期**定位虚拟机性能问题的常用工具。

命令格式：

```bash
$ jstat options vmid [interval[s|ms] [count]]
```

常用参数：

- options，要查询的虚拟机信息，主要分为三类：
  - 类加载
    - `-class` 监视类加载、卸载数量、总空间以及类加载所耗费的时间
  - 运行时数据区、GC
    - `-gccapacity` 查看 GC 情况和 JVM 各区的**容量**（字节）
    - `-gc` 查看 GC 情况和 JVM 各区的**容量**和**使用量**（字节）
    - `-gcutil` 查看 GC 情况和 JVM 各区的**使用率**（%）
    - ...
  - JIT
    - `-compiler` 输出即时编译器编译过的方法、耗时等信息
    - `-printcompilation` 输出已经被即时编译的方法
- vmid，如果是本地虚拟机进程，VMID 与 LVMID 一致；如果是远程虚拟机进程，则 VMID 的格式为：`[protocol:][//]lvmid[@hostname[:port]/servername]`
- interval，间隔时间，单位为秒或者毫秒
- count，打印次数，如果缺省则打印无数次

示例展示：

此示例连接到 lvmid 21891，并以 250 毫秒的间隔获取 7 个样本，每 6 行显示一次标题（`[-h<lines>]`），并显示由 `-gcutil` 选项指定的输出：

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

该示例结果显示，对象首先都在 Eden 区中创建，在第 3 和第 4 个样本之间由于 Eden 区装满，发生了 Young GC， gc 耗时 0.001 秒，并将对象从 Eden 区（E）提升到 Old 区（O），导致 Old 区的使用率从 9.49％ 增加到 9.51％。

 `-gcutil` 选项每列说明：

| 列名   | 描述                                  |
| ---- | ----------------------------------- |
| S0   | Heap 上的 Survivor space 0 区（单位**%**） |
| S1   | Heap 上的 Survivor space 1 区（单位**%**） |
| E    | Heap 上的 Eden space 区（单位**%**）       |
| O    | Heap 上的 Old space 区（单位**%**）        |
| P    | Perm space 区（单位**%**）               |

| 列名   | 描述                                     |
| ---- | -------------------------------------- |
| YGC  | 从应用程序启动到采样时发生 Young GC 的次数，**E 区满后触发** |
| YGCT | 从应用程序启动到采样时 Young GC 所用的时间（单位秒）        |
| FGC  | 从应用程序启动到采样时发生 Full GC 的次数， **O 区满后触发** |
| FGCT | 从应用程序启动到采样时 Full GC 所用的时间（单位秒）         |
| GCT  | 从应用程序启动到采样时用于垃圾回收的**总时间**（单位秒）         |

## jmap

`jmap` 命令用于生成虚拟机的内存转储快照（heap dump 文件），或查看堆内存信息。

常用参数：

* `-dump` 生成 Java 堆转储快照。格式为 `-dump:[live,]format=b,file=<filename>`，其中 `live` 子参数表示是否只 dump 出存活的对象。
* `-histo` 显示堆中对象统计信息，包括类、实例数量、合计容量。
* `-heap` 查看当前堆内存的详细信息，如配置信息 `Heap Configuration`、使用情况 `Heap Usage`。

```
$ jmap -heap 21090
Attaching to process ID 21090, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.79-b02

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio = 0
   MaxHeapFreeRatio = 100
   MaxHeapSize      = 3145728000 (3000.0MB)
   NewSize          = 2097152000 (2000.0MB)
   MaxNewSize       = 2097152000 (2000.0MB)
   OldSize          = 5439488 (5.1875MB)
   NewRatio         = 2
   SurvivorRatio    = 8
   PermSize         = 268435456 (256.0MB)  // JDK8+ MetaspaceSize
   MaxPermSize      = 268435456 (256.0MB)  // JDK8+ MaxMetaspaceSize
   G1HeapRegionSize = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 1762656256 (1681.0MB)
   used     = 1420607552 (1354.7969360351562MB)
   free     = 342048704 (326.20306396484375MB)
   80.59470172725499% used
From Space:
   capacity = 138412032 (132.0MB)
   used     = 0 (0.0MB)
   free     = 138412032 (132.0MB)
   0.0% used
To Space:
   capacity = 138412032 (132.0MB)
   used     = 0 (0.0MB)
   free     = 138412032 (132.0MB)
   0.0% used
PS Old Generation
   capacity = 1048576000 (1000.0MB)
   used     = 1048403072 (999.8350830078125MB)
   free     = 172928 (0.1649169921875MB)
   99.98350830078125% used
PS Perm Generation  // JDK8+ 没有该区域
   capacity = 268435456 (256.0MB)
   used     = 67917928 (64.7715835571289MB)
   free     = 200517528 (191.2284164428711MB)
   25.30139982700348% used
```

注意，由于此例中使用的 JDK 7 版本，因此 Heap 中包含 Perm Generation。如果使用的 JDK 8 以上版本，则 Heap 不再包含此区域，取而代之的是在 Heap 之外有一块 Metaspace。

例如上述例子通过 `jmap -heap pid` 命令发现了某个服务 O 区内存被占满的问题：`Old Generation` 达到 99.98350830078125% used，O 区内存被占满，可以通过 `jstack` 继续排查 JVM 内存的**动态使用情况**。

## jstack

`jstack` 命令用于查看当前线程堆栈信息，根据堆栈信息我们可以定位到具体代码，所以它在 JVM 性能调优中使用得非常多。

``` 
$ jstack 21090 > /tmp/localfile
$ less /tmp/localfile

Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.79-b02 mixed mode):

"Attach Listener" daemon prio=10 tid=0x00007f67e03b4800 nid=0x7bb9 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"catalina-exec-8000" daemon prio=10 tid=0x00007f67ba4a0000 nid=0x795a waiting on condition [0x00007f6558c0a000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007886ab360> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
	at org.apache.http.pool.PoolEntryFuture.await(PoolEntryFuture.java:139)
	at org.apache.http.pool.AbstractConnPool.getPoolEntryBlocking(AbstractConnPool.java:307)
	at org.apache.http.pool.AbstractConnPool.access$000(AbstractConnPool.java:65)
	at org.apache.http.pool.AbstractConnPool$2.getPoolEntry(AbstractConnPool.java:193)
	at org.apache.http.pool.AbstractConnPool$2.getPoolEntry(AbstractConnPool.java:186)
	at org.apache.http.pool.PoolEntryFuture.get(PoolEntryFuture.java:108)
	at org.apache.http.impl.conn.PoolingClientConnectionManager.leaseConnection(PoolingClientConnectionManager.java:212)
	at org.apache.http.impl.conn.PoolingClientConnectionManager$1.getConnection(PoolingClientConnectionManager.java:199)
	at org.apache.http.impl.client.DefaultRequestDirector.execute(DefaultRequestDirector.java:424)
	at org.apache.http.impl.client.AbstractHttpClient.doExecute(AbstractHttpClient.java:884)
	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:82)
	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:107)
	at com.xxx.xxx.xxx.HttpClientService.doPost(HttpClientService.java:103)
	......
```

该例子中导出的 `/tmp/localfile` 文件异常大，里面有大量 `java.lang.Thread.State: WAITING (parking)` 状态的线程，导致 O 区内存被占满，根据堆栈可以定位到具体的问题代码，可以初步判断是 HTTP 连接耗尽资源导致的问题。

# 参考

《深入理解 Java 虚拟机》