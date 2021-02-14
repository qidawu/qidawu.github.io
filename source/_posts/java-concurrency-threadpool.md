---
title: 并发编程系列（三）Java 线程池总结
date: 2020-05-03 22:07:11
updated: 
tags: [并发编程, Java]
typora-root-url: ..
---

为什么要用线程池？

> 线程的创建和销毁是有代价的。
>
> 如果请求的到达率非常高且请求的处理过程是轻量级的，那么为每个请求创建一个新线程将消耗大量的计算资源。
>
> 活跃的线程会消耗系统资源，尤其是内存。大量空闲线程会占用许多内存，给垃圾回收器带来压力，而且大量线程竞争 CPU 资源还会产生其它的性能开销。
>
> 可创建线程的数量上存在限制，如果创建太多线程，会使系统饱和甚至抛出 `OutOfMemoryException` 。

问题如下：

![no_thread_pool_design](/img/java/concurrent/no_thread_pool_design.png)

为了解决以上问题，从 Java 5 开始 JDK 并发 API 提供了 Executor Framework，用于**将任务的创建与执行分离**，避免使用者直接与 `Thread` 对象打交道，通过池化设计与阻塞队列保护系统资源：

![thread_pool_design](/img/java/concurrent/thread_pool_design.png)

使用 Executor Framework 的第一步就是创建一个 `ThreadPoolExecutor` 类的对象。你可以使用这个类提供的 **四个构造方法**或 `Executors` **工厂类**来创建 `ThreadPoolExecutor` 。一旦有了执行者，你就可以提交 `Runnable` 或 `Callable` 对象给执行者来执行。

# 自定义线程池

## 源码解析

`Executor` 接口的实现类如下：

![subtypes_of_Executor](/img/java/concurrent/subtypes_of_Executor.png)

其中，`ThreadPoolExecutor` 类实现了两个核心接口 `Executor` 和 `ExecutorService`，方法如下：

![ThreadPoolExecutor](/img/java/concurrent/Executor.png)

`ThreadPoolExecutor` 类的成员变量：

```java
/**
  * 线程池使用一个int变量存储线程池状态和工作线程数
  * int4个字节，32位，用高三位存储线程池状态，低29位存储工作线程数
  * 为什么使用一个变量来同时表示线程状态和线程数？就是节省空间。咨询了一下写c的朋友，他们经常这么写
  **/
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
//COUNT_BITS=29
private static final int COUNT_BITS = Integer.SIZE - 3;
//理论上线程池最大线程数量CAPACITY=(2^29)-1，即 536,870,911
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

//获取线程池状态
private static int runStateOf(int c)     { return c & ~CAPACITY; }
//获取工作线程数
private static int workerCountOf(int c)  { return c & CAPACITY; }
//初始化ctl
private static int ctlOf(int rs, int wc) { return rs | wc; }

/**
  * 线程池状态转换
  * RUNNING -> SHUTDOWN
  * RUNNING or SHUTDOWN -> STOP
  * SHUTDOWN or STOP -> TIDYING
  * TIDYING -> TERMINATED  terminated()执行完后变为该TERMINATED
  */
//接受新任务，可以处理阻塞队列里的任务
private static final int RUNNING    = -1 << COUNT_BITS;
//不接受新任务，可以处理阻塞队列里的任务。执行shutdown()会变为SHUTDOWN
private static final int SHUTDOWN   =  0 << COUNT_BITS;
//不接受新的任务，不处理阻塞队列里的任务，中断正在处理的任务。执行shutdownNow()会变为STOP
private static final int STOP       =  1 << COUNT_BITS;
//临时过渡状态，所有的任务都执行完了，当前线程池有效的线程数量为0，这个时候线程池的状态是TIDYING，执行terminated()变为TERMINATED
private static final int TIDYING    =  2 << COUNT_BITS;
//终止状态，terminated()调用完成后的状态
private static final int TERMINATED =  3 << COUNT_BITS;

//重入锁，更新线程池核心大小、线程池最大大小等都有用到
private final ReentrantLock mainLock = new ReentrantLock();
//用于存储woker
private final HashSet<Worker> workers = new HashSet<Worker>();
//用于终止线程池
private final Condition termination = mainLock.newCondition();
//记录线程池中曾经出现过的最大线程数
private int largestPoolSize;
//完成任务数量
private long completedTaskCount;   

/**
 * 核心线程数
 * 核心线程会一直存活，即使没有任务需要处理，当线程数小于核心线程数时。
 * 即使现有的线程空闲，线程池也会优先创建新线程来处理任务，而不是直接交给现有的线程处理。
 * 核心线程数在初始化时不会创建，只有提交任务的时候才会创建。核心线程在allowCoreThreadTimeout为true的时候超时会退出。
 */
private volatile int corePoolSize;
 /** 最大线程数
   * 当线程数大于或者等于核心线程，且任务队列已满时，线程池会创建新的线程，直到线程数量达到maxPoolSize。
   * 如果线程数已等于maxPoolSize，且任务队列已满，则已超出线程池的处理能力，线程池会采取拒绝操作。
   */
private volatile int maximumPoolSize;
/**
  * 线程空闲时间
  * 当线程空闲时间达到keepAliveTime，该线程会退出，直到线程数量等于corePoolSize。
  * 如果allowCoreThreadTimeout设置为true，则所有线程均会退出。
  */
private volatile long keepAliveTime;
//是否允许核心线程空闲超时退出，默认值为false。
private volatile boolean allowCoreThreadTimeOut;
//线程工厂
private volatile ThreadFactory threadFactory;
//用于保存等待执行的任务的阻塞队列。比如LinkedBlockQueue，SynchronousQueue等
private final BlockingQueue<Runnable> workQueue;
/**
 *  rejectedExecutionHandler：任务拒绝策略
 *  DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务
 *  AbortPolicy：抛出异常。这也是默认的策略
 *  CallerRunsPolicy：用调用者所在线程来运行任务
 *  DiscardPolicy：不处理，丢弃掉
 */
private volatile RejectedExecutionHandler handler;
//默认的拒绝策略：抛出异常
private static final RejectedExecutionHandler defaultHandler =
    new AbortPolicy();
private static final RuntimePermission shutdownPerm =
    new RuntimePermission("modifyThread");
```

`ThreadPoolExecutor` 提供了四个构造方法，以参数最多的为例：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    // 参数校验
    if (corePoolSize < 0 || maximumPoolSize <= 0 || maximumPoolSize < corePoolSize || keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();

    this.acc = System.getSecurityManager() == null ? null : AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

下面分别介绍源码中涉及的重要属性。

## 重要属性

作为一个线程池，有两个关键属性：

* 线程池状态 `runState`
* 工作线程数 `workerCnt`

这两个关键属性保存在名为 `ctl` 的 `AtomicInteger` 类型属性之中，高 3 位表示 `runState`，低 29 位表示 `workerCnt`，如下：

![ctl](/img/java/concurrent/ctl.png)

为什么要用 3 位来表示线程池的状态呢，原因是线程池一共有 5 种状态，而 2 位只能表示出 4 种情况，所以至少需要 3 位才能表示得了 5 种状态，如下：

```
runState workerCnt                       runState workerCnt
     000 00000000000000000000000000000   SHUTDOWN empty
‭‭     001 00000000000000000000000000000       STOP empty
     010 00000000000000000000000000000    TIDYING empty
     ‭011 00000000000000000000000000000‬ TERMINATED empty
     111 00000000000000000000000000000    RUNNING empty
‭     111 11111111111111111111111111111    RUNNING full
```

通过 `ctlOf` 方法初始化 `ctl` 属性：

```Java
// 初始化ctl
private static int ctlOf(int rs, int wc) { return rs | wc; }

// 或运算符(|)规则：1|1=1
//                 1|0=1
//                 0|1=1
//                 0|0=0
// 以初始化参数 ctlOf(RUNNING, 0) 为例：
  11100000000000000000000000000000
| 00000000000000000000000000000000
= 11100000000000000000000000000000
```

通过 `runStateOf` 方法获取线程池状态 `runState`：

```Java
// 获取线程池状态
private static int runStateOf(int c)     { return c & ~CAPACITY; }

// 取反运算符(~)规则：~1=0
//                   ~0=1
// 以 c = 111 11111111111111111111111111111（RUNNING full）为例：
     ~11111111111111111111111111111
=     00000000000000000000000000000
& 111 11111111111111111111111111111
= 111
```

通过 `workerCountOf` 方法获取工作线程数 `workerCnt`：

```Java
// 获取工作线程数
private static int workerCountOf(int c)  { return c & CAPACITY; }

// 与运算符(&)规则：1&1=1
//                 1&0=0
//                 0&1=0
//                 0&0=0
// 以 c = 111 11111111111111111111111111111（RUNNING full）为例：
  111 11111111111111111111111111111
&     11111111111111111111111111111
=     11111111111111111111111111111
```

### 线程池状态

线程池状态用于标识线程池内部的一些运行情况，线程池的开启到关闭的过程就是线程池状态的一个流转的过程。

线程池共有五种状态：

![run_state](/img/java/concurrent/run_state.png)

| 状态         | `runState` | 含义                                                         |
| :----------- | ---------- | :----------------------------------------------------------- |
| `RUNNING`    | 111        | 运行状态，该状态下线程池可以接受新的任务，也可以处理阻塞队列中的任务。<br/>执行 `shutdown` 方法可进入 `SHUTDOWN` 状态。<br/>执行 `shutdownNow` 方法可进入 `STOP` 状态。 |
| `SHUTDOWN`   | 000        | 待关闭状态，不再接受新的任务，继续处理阻塞队列中的任务。<br/>当阻塞队列中的任务为空，并且工作线程数为 0 时，进入 `TIDYING` 状态。 |
| `STOP`       | 001        | 停止状态，不接收新任务，也不处理阻塞队列中的任务，并且会尝试结束执行中的任务。<br/>当工作线程数为 0 时，进入 `TIDYING` 状态。 |
| `TIDYING`    | 010        | 整理状态，此时任务都已经执行完毕，并且也没有工作线程 执行 `terminated` 方法后进入 `TERMINATED` 状态。 |
| `TERMINATED` | 011        | 终止状态，此时线程池完全终止了，并完成了所有资源的释放。     |

### 工作线程数

尽管理论上线程池最大线程数量可达 `CAPACITY` 数，但是实际上都会通过 `maximumPoolSize` 限制最大线程数。因此工作线程数 `workerCnt` 的个数可能在 0 至 `maximumPoolSize` 之间变化。

当工作线程的空闲时间达到 `keepAliveTime`，该工作线程会退出，直到工作线程数 `workerCnt` 等于 `corePoolSize`。如果 `allowCoreThreadTimeout` 设置为 `true`，则所有工作线程均会退出。

![worker_count](/img/java/concurrent/worker_count.png)

注意：

- 整个线程池的基本执行过程：创建核心线程（Core Thread） > 任务排队 > 创建临时线程（Temp Thread）。
- 如果将 `maximumPoolSize` 设置为基本的无界值（如 Integer.MAX_VALUE），可能会创建大量的线程，从而导致 OOM。因此要限定 `maximumPoolSize` 的大小。
- 如果将 `corePoolSize` 和 `maximumPoolSize` 设置为相同值，则创建了 Fixed 固定大小的线程池。

### 线程工厂

通过提供不同的 `ThreadFactory` 接口实现，可以改变被创建线程 `Thread` 的名称、线程组、优先级、守护进程状态，等等。

参考《[Java 并发编程系列（一）常用包总结](/2018/07/10/java-concurrent-package/#ThreadFactory)》

### 阻塞队列

阻塞队列的使用详见另一篇《[Java 集合框架系列（三）并发实现总结](/2018/04/21/java-collections-framework-concurrent-impl/)》。

![work_queue](/img/java/concurrent/work_queue.png)

### 拒绝策略

拒绝策略，默认有四种实现：

* `AbortPolicy`：抛出异常，默认的策略。
* `DiscardPolicy`：不处理，丢弃掉。
* `DiscardOldestPolicy`：丢弃队列中最近的一个任务，并执行该任务。
* `CallerRunsPolicy`：用调用者所在线程来执行该任务。

![RejectedExecutionHandler](/img/java/concurrent/RejectedExecutionHandler.png)

通过 `RejectedExecutionHandler` 接口可以实现更多策略，例如记录日志或持久化不能处理的任务，或者发出告警。

```java
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

## 执行流程

`execute` 方法的整体执行流程如下：

```
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
```

![work_flow_of_execute_method](/img/java/concurrent/work_flow_of_execute_method.png)

# 使用工厂类创建线程池

`java.util.concurrent.ThreadPoolExecutor` 提供了四个不同的构造方法，但由于它们的复杂性（参数较多），Java 并发 API 提供了 `java.util.concurrent.Executors` 工厂类来简化线程池的构造，常用方法如下：

```java
// 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
public static ExecutorService newFixedThreadPool(...) {...}
// 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序（FIFO, LIFO, 优先级）执行。
public static ExecutorService newSingleThreadExecutor(...) {...}
// 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
public static ExecutorService newCachedThreadPool(...) {...}
// 创建一个定长线程池，支持定时及周期性任务执行。
public static ScheduledExecutorService newScheduledThreadPool(...) {...}
```

但是这种方式并不推荐使用，参考《阿里巴巴 Java 开发手册》：	

![principal of executors](/img/java/concurrent/principal_of_executors.png)

`java.util.concurrent.Executors` 源码分析如下，首先是 `newFixedThreadPool(...)` 和 `newSingleThreadExecutor(...)`：

```java
// Fixed 限定 corePoolSize 和 maximumPoolSize 为相同大小，即线程池大小固定（意味着无法扩展）
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

// Single 其实就是 Fixed 为 1 的变种
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

上述方法中，关键在于对 `java.util.concurrent.LinkedBlockingQueue` 的构造，使用了默认的无参构造方法：

```java
// 允许的请求队列长度（capacity）为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}
```

然后是 `newCachedThreadPool(...)` 和 `newScheduledThreadPool(...)`：

```java
// 允许的创建线程数量（maximumPoolSize）为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

// 问题在于 ScheduledThreadPoolExecutor 构造方法的默认参数
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

看下 `java.util.concurrent.ScheduledThreadPoolExecutor` 的构造方法：

```java
// ScheduledThreadPoolExecutor 构造方法中，允许的创建线程数量（maximumPoolSize）为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```
