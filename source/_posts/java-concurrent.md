---
title: Java 并发编程类总结
date: 2019-02-24 15:07:11
updated: 
tags: Java
---

工作中常用到一些并发编程类，这里做一些总结。

# JDK 包简介

JDK 中涉及到线程的包如下：

## java.lang

> 内含基础并发类。

### Runnable

异步任务需实现的接口。

### Thread

程序中的执行线程。

`Thread` 对象中保存了一些**属性**能够帮助我们来辨别每一个线程，知道它的状态，调整控制其优先级等：

`ID`

> 每个线程的独特标识。

`Name`

> 线程的名称。

`Priority`

> 线程对象的优先级。优先级别在 1-10 之间，1 是最低级，10 是最高级。不建议改变它们的优先级。

`Daemon`

> 是否为守护线程。
>
> Java 有一种特别的线程叫做守护线程。这种线程的**优先级非常低**，通常在程序里没有其他线程运行时才会执行它。当守护线程是程序里唯一在运行的线程时，JVM 会结束守护线程并终止程序。
>
> 根据这些特点，守护线程通常用于在同一程序里给普通线程（也叫使用者线程）提供服务。它们通常无限循环的等待服务请求或执行线程任务。它们不能做重要的任务，因为我们不知道什么时候会被分配到 CPU 时间片，并且只要没有其他线程在运行，它们可能随时被终止。**JAVA中最典型的这种类型代表就是垃圾回收器 GC**。
>
> 只能在 start() 方法之前可以调用 setDaemon() 方法。一旦线程运行了，就不能修改守护状态。
>
> 可以使用 isDaemon() 方法来检查线程是否是守护线程。

`Thread.State`

> 线程的状态，共六种：
> NEW
> RUNNABLE
> BLOCKED
> WAITING
> TIME_WAITING
> TERMINATED

`Thread.UncaughtExceptionHandler`

> 用于捕获和处理线程对象抛出的 Unchecked Exception 来避免程序终结。

`Thread` 类提供了以下几类**方法**：

* 线程协作
* 线程中断
* 线程让步
* 线程睡眠
* 线程合并
* ……

### ThreadLocal<T>

`ThreadLocal` 存放的值是线程内共享的，线程间互斥的，主要用于在线程内共享一些数据。

### ThreadGroup

## java.util.concurrent

> JDK 5 引入的 Executor Framework ，用于取代传统的并发编程。

![Package concurrent](/img/java/concurrent/package_concurrent.png)

核心接口：

```java
public interface Executor {
    void execute(Runnable command);
}
```

```java
public interface ExecutorService extends Executor {
    void shutdown();
    List<Runnable> shutdownNow();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

`java.util.concurrent.BlockingQueue`：

![BlockingQueue](/img/java/concurrent/BlockingQueue.png)

```java
public interface BlockingQueue<E> extends Queue<E> {
    boolean add(E e);
    boolean offer(E e);
    void put(E e) throws InterruptedException;
    boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;
    E take() throws InterruptedException;
    E poll(long timeout, TimeUnit unit) throws InterruptedException;
    int remainingCapacity();
    boolean remove(Object o);
    public boolean contains(Object o);
    int drainTo(Collection<? super E> c);
    int drainTo(Collection<? super E> c, int maxElements);
}
```



## java.util.concurrent.locks

> 用于实现线程安全与通信。

![Package locks](/img/java/concurrent/package_locks.png)

## java.util.concurrent.atomic

> 使用这些数据结构可以避免在并发程序中使用同步代码块。

![Package atomic](/img/java/concurrent/package_atomic.png)

# Spring 包简介

## org.springframework.scheduling

Spring Framework 中并发编程相关的类主要位于 `spring-context` 下的 `org.springframework.scheduling`，例如其子包 `concurrent`：

![org.springframework.scheduling.concurrent](/img/java/concurrent/package_spring_concurrent.png)

此外，`org.springframework.scheduling.concurrent.CustomizableThreadFactory` 还：

* 实现了 `java.util.concurrent.ThreadFactory` 线程工厂接口，源码如下：

  ```java
  // Executors.defaultThreadFactory 方法提供了一个实用的简单实现，为新线程设置了上下文，详见源码
  public interface ThreadFactory {
      Thread newThread(Runnable r);
  }
  ```

* 继承了 `org.springframework.util.CustomizableThreadFactory` 类，用于创建新线程，并提供各种线程属性自定义配置（如线程名前缀、线程优先级等）

![org.springframework.util.CustomizableThreadFactory](/img/java/concurrent/CustomizableThreadFactory.png)

# 两种异步任务

## 无返回结果的异步任务

使用常规的 `java.lang.Runnable` 。

## 有返回结果的异步任务

Executor Framework 的一个重要优点是提供了 `java.util.concurrent.Callable<V>` 接口用于返回异步任务的结果。它的用法跟 `Runnable` 接口很相似，但它提供了两种改进：

* 这个接口中主要的方法叫 `call()` ，可以返回结果。
* 当你提交 `Callable` 对象到 `Executor` 执行者，你可以获取一个实现 `Future` 接口的对象，你可以用这个对象来控制和获取 `Callable` 对象的状态和结果。

# 线程池

为什么要用线程池？

> 线程的创建和销毁是有代价的。
>
> 如果请求的到达率非常高且请求的处理过程是轻量级的，那么为每个请求创建一个新线程将消耗大量的计算资源。
>
> 活跃的线程会消耗系统资源，尤其是内存。如果可运行线程数量多于可用处理器数量，则有些线程会被闲置；大量空闲线程会占用许多内存，给垃圾回收器带来压力，而且大量线程竞争 CPU 资源还会产生其它的性能开销。
>
> 可创建线程的数量上存在限制，如果创建太多线程，会使系统饱和甚至抛出 `OutOfMemoryException` 。

为了解决以上问题，从 Java 5 开始 JDK 并发 API 提供了 Executor Framework。核心接口是 `Executor` ，其子接口是 `ExecutorService` ，而 `ThreadPoolExecutor` 类则实现了这两个接口。

Executor Framework 主要用于**将任务的创建与执行分离**，避免使用者直接与万恶的 `Thread` 对象打交道。

使用 Executor Framework 的第一步就是创建一个 `ThreadPoolExecutor` 类的对象。你可以使用这个类提供的 **四个构造方法**或 **Executors 工厂类**来创建 `ThreadPoolExecutor` 。一旦有了执行者，你就可以提交 `Runnable` 或 `Callable` 对象给执行者来执行。

## 使用 Executors 工厂类构造线程池

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

## 使用 ThreadPoolExecutor 构造方法定制线程池

以参数最多的构造方法为例，理解下各参数的用途：

```java
ThreadPoolExecutor(
    int corePoolSize, 
    int maximumPoolSize, 
    long keepAliveTime, 
    TimeUnit unit, 
    BlockingQueue<Runnable> workQueue, 
    ThreadFactory threadFactory, 
    RejectedExecutionHandler handler) 
```

![ThreadPoolExecutor](/img/java/concurrent/threadpoolexecutor.png)

注意点：

- 整个线程池的基本执行过程：创建初始线程 > 线程排队 > 创建扩容线程。
- `corePoolSize` 用于设置线程池中需保留线程数，即使该线程处于 idle 闲置状态。
- 如果将 `maximumPoolSize` 设置为基本的无界值（如 Integer.MAX_VALUE），则允许池适应任意数量的并发任务，可能会创建大量的线程，从而导致 OOM。因此要限定 `maximumPoolSize` 的大小。
- 如果将 `corePoolSize` 和 `maximumPoolSize` 设置为相同值，则创建了 Fixed 固定大小的线程池。
- 通过提供不同的 `ThreadFactory` 接口实现，可以改变被创建线程的名称、线程组、优先级、守护进程状态，等等。
- 任务排队有几种通用策略，通过 `BlockingQueue` 接口可以实现更多策略。
  - `java.util.concurrent.LinkedBlockingQueue`，默认用于 `newFixedThreadPool(...)` 和 `newSingleThreadExecutor(...)`
  - `java.util.concurrent.SynchronousQueue`，默认用于 `newCachedThreadPool(...)`
  - `java.util.concurrent.DelayedWorkQueue`，默认用于 `newScheduledThreadPool(...)`
- 任务拒绝有四种预定义策略，通过 `RejectedExecutionHandler` 接口可以实现更多策略。

# 最后

本文暂不涉及：

* 线程同步/锁的理论知识和 API 操作，这是一个很大的话题，需要后续另起一篇来总结。
* 第三方并发工具或框架，如 Apache Commoms Lang 的 Concurrent 部分，如 Spring Framework 的一些并发类，使用它们可以简化并发编程。