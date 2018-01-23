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

# jmap

`jmap` (Memory Map) 命令用于查看堆内存。

常用参数：

- `-heap` 查看当前堆内存的配置、各区使用情况，以及 GC 算法。

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
   PermSize         = 268435456 (256.0MB)
   MaxPermSize      = 268435456 (256.0MB)
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
PS Perm Generation
   capacity = 268435456 (256.0MB)
   used     = 67917928 (64.7715835571289MB)
   free     = 200517528 (191.2284164428711MB)
   25.30139982700348% used
```

各区描述如下：

![JVM 堆内存布局](/img/java/jvm_space.jpg)

堆内存 = 新生代（Eden 区 + 两个 Survivor 区（From、To）） + 老年代 + 永久代

Heap = Young Generation(Eden Space + From Space + To Space) + Old Generation + Perm Generation

Tomcat 参数可以调整 JVM 各区：

| 参数                               | 描述                                       |
| -------------------------------- | ---------------------------------------- |
| `-Xss`                           | 设置每个线程可使用的内存大小                           |
| `-Xms`、`-Xmx`                    | Heap 堆（S0+S1+E+O）的初始值和最大值，Server 端 JVM 建议将 `-Xms` 和 `-Xmx` 设为相同值； |
| `-Xmn`                           | Heap 堆内新生代（S0+S1+E）的大小，通过这个值我们也可以得到老年代的大小：`-Xmx` 减去 `-Xmn`； 设置 `-Xmn` 的效果等同于设置了相同的 `-XX:NewSize` 和 `-XX:MaxNewSize`。 |
| `-XX:NewSize`、`-XX:MaxNewSize`   | Heap 堆内新生代（S0+S1+E）的初始值和最大值，新生代与老年代的大小比例默认为 2:1； |
| `-XX:NewRatio`                   | 设置新生代和老年代的比值，例如该值为3，则表示新生代和老年代比值为1:3。    |
| ` -XX:SurvivorRatio`             | 设置新生代中 E 区和 S 区的比例， 即 -XX:SurvivorRatio=eden/s0=eden/s1。 |
| `-XX:PermSize`、`-XX:MaxPermSize` | Perm 初始值和最大值；                            |

例如上述例子通过 `jmap -heap pid` 命令发现了某个服务 O 区内存被占满的问题：`Old Generation` 达到 99.98350830078125% used，O 区内存被占满，可以通过 `jstack` 继续排查问题。

# jstat

`jstat` (JVM Statistics Monitoring Tool) 命令用于监控当前 JVM 的性能统计信息。

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

该示例结果显示，对象首先都在 Eden 区中创建，在第 3 和第 4 个样本之间由于 Eden 区装满，发生了 Young GC。 gc 耗时 0.001 秒，并将对象从 Eden 区（E）提升到 Old 区（O），导致 Old 区的使用率从 9.49％ 增加到 9.51％。

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

# jstack

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

《[JVM性能调优监控工具jps、jstack、jmap、jhat、jstat、hprof使用详解](https://my.oschina.net/feichexia/blog/196575)》