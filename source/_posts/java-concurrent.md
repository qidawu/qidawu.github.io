---
title: "Java 并发编程类总结"
date: 2016-05-01 15:07:11
updated: 
tags: Java
description: "工作中常用到一些并发编程类，在此做了一些简单、系统的总结。"
---

工作中常用到一些并发编程类，在此做了一些简单、系统的总结。

# JDK 包简介

JDK 中涉及到线程的包如下：

java.lang

> 内含基础并发类。

java.util.concurrent

> JDK 5 引入的 Executor Framework ，用于取代传统的并发编程。

java.util.concurrent.locks

> 用于实现线程安全与通信。

java.util.concurrent.atomic

> 使用这些数据结构可以避免在并发程序中使用同步代码块。

# java.lang 基础类

## Runnable

异步任务需实现的接口。

## Thread

程序中的执行线程。

### 属性

`Thread` 对象中保存了一些属性能够帮助我们来辨别每一个线程，知道它的状态，调整控制其优先级等。

ID 

> 每个线程的独特标识。

Name

> 线程的名称。

Priority

> 线程对象的优先级。优先级别在 1-10 之间，1 是最低级，10 是最高级。不建议改变它们的优先级。

Daemon

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

### 方法

`Thread` 类提供了以下几类方法：

* 线程协作
* 线程中断
* 线程让步
* 线程睡眠
* 线程合并
* ……

## ThreadLocal<T>

`ThreadLocal` 存放的值是线程内共享的，线程间互斥的，主要用于在线程内共享一些数据。

## ThreadGroup

# java.util.concurrent

## 两种异步任务

### 无返回结果的异步任务

使用常规的 `java.lang.Runnable` 。

### 有返回结果的异步任务

Executor Framework 的一个重要优点是提供了 `java.util.concurrent.Callable<V>` 接口用于返回异步任务的结果。它的用法跟 `Runnable` 接口很相似，但它提供了两种改进：

* 这个接口中主要的方法叫 `call()` ，可以返回结果。
* 当你提交 `Callable` 对象到 `Executor` 执行者，你可以获取一个实现 `Future` 接口的对象，你可以用这个对象来控制和获取 `Callable` 对象的状态和结果。

## 线程池

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

### 使用 Executors 工厂类构造线程池

java.util.concurrent.Executors

> ThreadPoolExecutor 虽然有四个不同的构造方法，但由于它们的复杂性（参数较多），Java 并发 API 提供 Executors 工厂类来构造执行者和其他相关对象，推荐使用。

常用方法：

```
newCachedThreadPool(...)
newFixedThreadPool(...)
newSingleThreadExecutor(...)
newScheduledThreadPool(...)
```

### 使用构造方法定制线程池

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

![ThreadPoolExecutor](https://github.com/cynthia903/study-notes/blob/master/Programming/Java/assets/java_threadpoolexecutor.png)

注意点：

- 整个线程池的基本执行过程：创建初始线程 > 线程排队 > 创建扩容线程
- 如果设置的 corePoolSize 和 maximumPoolSize 相同，则创建了固定大小的线程池。
- 如果将 maximumPoolSize 设置为基本的无界值（如 Integer.MAX_VALUE），则允许池适应任意数量的并发任务。
- 通过提供不同的 `ThreadFactory` 接口实现，可以改变被创建线程的名称、线程组、优先级、守护进程状态，等等。
- 任务排队有三种通用策略，通过 `BlockingQueue` 接口可以实现更多策略。
- 任务拒绝有四种预定义策略，通过 `RejectedExecutionHandler` 接口可以实现更多策略。

# 最后

本文暂不涉及：

* 线程同步/锁的理论知识和 API 操作，这是一个很大的话题，需要后续另起一篇来总结。
* 第三方并发工具或框架，如 Apache Commoms Lang 的 Concurrent 部分，如 Spring Framework 的一些并发类，使用它们可以简化并发编程。