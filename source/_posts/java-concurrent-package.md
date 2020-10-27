---
title: Java 并发编程系列（一）常用包总结
date: 2018-07-10 15:07:11
updated: 
tags: Java
typora-root-url: ..
---

工作中常用到一些并发编程类，这里做一些总结。

JDK 中涉及到线程的包如下：

# java.lang

> 内含基础并发类。

## Runnable

无返回结果的异步任务。

## Thread

程序中的执行线程。

### 属性

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

`Thread.UncaughtExceptionHandler`

> 用于捕获和处理线程对象抛出的 Unchecked Exception 来避免程序终结。

`Thread.State`

> 线程的状态，共六种：
> NEW
> RUNNABLE
> BLOCKED
> WAITING
> TIME_WAITING
> TERMINATED

### 方法

`Thread` 类提供了以下几类**方法**：

* 线程睡眠 `Thread.sleep(...)`
* 线程中断 `Thread.interrupt()`
* 线程让步 `Thread.yield()`
* 线程合并 `Thread.join(...)`
* ……

`Object` 提供了一组线程协作方法：

* 线程协作 `Object.wait/notify`

## ThreadLocal<T>

`ThreadLocal` 存放的值是线程内共享的，线程间互斥的，主要用于在线程内共享一些数据。

可以通过实现 `AutoCloseable` 以使用 try-with-resources 语法简化 `ThreadLocal` 资源清理：

```java
try (ChannelContext ctx = new ChannelContext(channel)) {
    ...
}
```

实现如下：

```java
@Slf4j
public class ChannelContext implements AutoCloseable {

    private static final ThreadLocal<Channel> CTX = new ThreadLocal<>();

    public ChannelContext(FundChannelDTO dto) {
        Channel channel = Channel.builder()
                .appId(dto.getAppId().toString())
                .build();
        CTX.set(channel);
    }

    public ChannelContext(Channel channel) {
        CTX.set(channel);
    }

    public static Channel get() {
        return CTX.get();
    }

    @Override
    public void close() {
        try {
            CTX.remove();
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
    }

    @Getter
    @Builder
    static class Channel {
        private final String appId;
    }

}
```

## ThreadGroup

# java.util

## TimerTask

`Timer` 是 JDK 中提供的一个定时器工具类，使用的时候会在主线程之外起一个单独的线程执行指定的定时任务 `TimerTask`，可以指定执行一次或者反复执行多次。

`TimerTask` 是一个实现了 `Runnable` 接口的抽象类，代表一个可以被 `Timer` 执行的任务。

![TimerTask](/img/java/concurrent/TimerTask.png)

# java.util.concurrent

> JDK 5 引入的 Executor Framework ，用于取代传统的并发编程。

![Package concurrent](/img/java/concurrent/package_concurrent.png)

## ThreadFactory

通过提供不同的 `ThreadFactory` 接口实现，可以改变被创建线程 `Thread` 的**属性**。`ThreadFactory` 有几种创建方式：

1、完全自定义方式。缺点是需要在 `newThread` 方法中实现的代码较多：

```java
ThreadFactory threadFactory = runnable -> {
    Thread thread = new Thread(runnable);
    thread.setName("...");
    return thread;
};
```

2、使用 `Executors` 工具类，这也是 `Executors` 工具类中提供的几种默认线程池所使用的方式。

缺点是只能使用默认属性，无法修改：

```java
ThreadFactory threadFactory = Executors.defaultThreadFactory();
```

优点是实现了基本的线程名称自增，该实现如下：

```java
    /**
     * The default thread factory
     */
    static class DefaultThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        DefaultThreadFactory() {
            SecurityManager s = System.getSecurityManager();
            group = (s != null) ? s.getThreadGroup() :
                                  Thread.currentThread().getThreadGroup();
            namePrefix = "pool-" +
                          poolNumber.getAndIncrement() +
                         "-thread-";
        }

        public Thread newThread(Runnable r) {
            Thread t = new Thread(group, r,
                                  namePrefix + threadNumber.getAndIncrement(),
                                  0);
            if (t.isDaemon())
                t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY)
                t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    }
```

3、通过 Guava 提供的 `ThreadFactoryBuilder`。优点是可以轻易自定义任何属性：

```java
ThreadFactory threadFactory = new ThreadFactoryBuilder().setNameFormat("wechat-notify-%d").build()
```

该实现如下，如果未提供自定义的 `ThreadFactory`，将使用 `Executors` 工具类提供的默认 `ThreadFactory` 并进行二次修改：

```java
  private static ThreadFactory build(ThreadFactoryBuilder builder) {
    final String nameFormat = builder.nameFormat;
    final Boolean daemon = builder.daemon;
    final Integer priority = builder.priority;
    final UncaughtExceptionHandler uncaughtExceptionHandler = builder.uncaughtExceptionHandler;
    final ThreadFactory backingThreadFactory = (builder.backingThreadFactory != null)
            ? builder.backingThreadFactory
            : Executors.defaultThreadFactory();
    final AtomicLong count = (nameFormat != null) ? new AtomicLong(0) : null;

    return new ThreadFactory() {
      @Override
      public Thread newThread(Runnable runnable) {
        Thread thread = backingThreadFactory.newThread(runnable);
        if (nameFormat != null) {
          thread.setName(format(nameFormat, count.getAndIncrement()));
        }
        if (daemon != null) {
          thread.setDaemon(daemon);
        }
        if (priority != null) {
          thread.setPriority(priority);
        }
        if (uncaughtExceptionHandler != null) {
          thread.setUncaughtExceptionHandler(uncaughtExceptionHandler);
        }
        return thread;
      }
    };
  }
```

## Callable

有返回结果的异步任务。Executor Framework 的一个重要优点是提供了 `java.util.concurrent.Callable<V>` 接口用于返回异步任务的结果。它的用法跟 `Runnable` 接口很相似，但它提供了两种改进：

* 这个接口中主要的方法叫 `call()` ，可以返回结果。
* 当你提交 `Callable` 对象到 `Executor` 执行者，你可以获取一个实现 `Future` 接口的对象，你可以用这个对象来控制和获取 `Callable` 对象的状态和结果。

## 工具类

`CountDownLatch`

`CyclicBarrier`

`Phaser`

`CompletableFuture`



`Semaphore`



`Exchanger`



`Executors`

## 线程池

参考另一篇《Java 并发编程系列（二）线程池总结》。

## 并发集合

详见另一个篇《Java 集合框架系列（三）并发实现总结》。

## 显式锁

java.util.concurrent.locks

> 用于实现线程安全与通信。

![Package locks](/img/java/concurrent/package_locks.png)

## 原子类

java.util.concurrent.atomic

> 使用这些数据结构可以避免在并发程序中使用同步代码块（synchronized 或 Lock）。

![Package atomic](/img/java/concurrent/package_atomic.png)

JDK 5 新增的原子类，底层基于魔术类 `Unsafe` 进行 CAS 无锁操作。实现类按功能分组如下：

|                | Integer                     | Long                     | Boolean         | 引用类型                                                     |
| -------------- | --------------------------- | ------------------------ | --------------- | ------------------------------------------------------------ |
| 基本类         | `AtomicInteger`             | `AtomicLong`             | `AtomicBoolean` |                                                              |
| 引用类型       |                             |                          |                 | `AtomicReference`<br/>`AtomicStampedReference`<br/>`AtomicMarkableReference` |
| 数组类型       | `AtomicIntegerArray`        | `AtomicLongArray`        |                 | `AtomicReferenceArray`                                       |
| 属性原子修改器 | `AtomicIntegerFieldUpdater` | `AtomicLongFieldUpdater` |                 | `AtomicReferenceFieldUpdater`                                |

JDK 8 新增 `Striped64` 累加计数器这个并发组件，64 指的是计数 64 bit 的数，即 `Long` 和 `Double` 类型。其实现类如下：

| Long              | Double              |
| ----------------- | ------------------- |
| `LongAdder`       | `DoubleAdder`       |
| `LongAccumulator` | `DoubleAccumulator` |

性能对比参考：http://www.manongjc.com/article/105666.html

# Spring 包简介

## org.springframework.scheduling

Spring Framework 中并发编程相关的类主要位于 `spring-context` 下的 `org.springframework.scheduling`，例如其子包 `concurrent`：

![org.springframework.scheduling.concurrent](/img/java/concurrent/package_spring_concurrent.png)

其中，顶层的 `org.springframework.scheduling.concurrent.CustomizableThreadFactory` 结构如下：

![org.springframework.util.CustomizableThreadFactory](/img/java/concurrent/CustomizableThreadFactory.png)

* `CustomizableThreadFactory` 实现了 `java.util.concurrent.ThreadFactory` 线程工厂接口，源码如下：

  ```java
  // Executors.defaultThreadFactory 方法提供了一个实用的简单实现，为新线程设置了上下文，详见源码
  public interface ThreadFactory {
      Thread newThread(Runnable r);
  }
  ```

* `CustomizableThreadFactory` 继承了 `org.springframework.util.CustomizableThreadCreator` 类，用于创建新线程，并提供各种线程属性自定义配置（如线程名前缀、线程优先级等）。

然后重点看下最常用的 `org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor` 类，提供的方法列表如下：

![ThreadPoolTaskExecutor 方法列表](/img/java/concurrent/ThreadPoolTaskExecutor.png)

当我们在实例化 `ThreadPoolTaskExecutor` 类的时候，其调用堆栈如下：

![](/img/java/concurrent/ThreadPoolTaskExecutor_threads.png)

可见，实际上是先调用了抽象父类 `ExecutorConfigurationSupport` 的 `afterPropertiesSet()` 和 `initialize()` 方法，最后再调用 `ThreadPoolTaskExecutor#initializeExecutor(...)`，该方法源码如下：

```java
    @Override
    protected ExecutorService initializeExecutor(
            ThreadFactory threadFactory, RejectedExecutionHandler rejectedExecutionHandler) {

        BlockingQueue<Runnable> queue = createQueue(this.queueCapacity);

        ThreadPoolExecutor executor;
        if (this.taskDecorator != null) {
            executor = new ThreadPoolExecutor(
                    this.corePoolSize, this.maxPoolSize, this.keepAliveSeconds, TimeUnit.SECONDS,
                    queue, threadFactory, rejectedExecutionHandler) {
                @Override
                public void execute(Runnable command) {
                    super.execute(taskDecorator.decorate(command));
                }
            };
        }
        else {
            executor = new ThreadPoolExecutor(
                    this.corePoolSize, this.maxPoolSize, this.keepAliveSeconds, TimeUnit.SECONDS,
                    queue, threadFactory, rejectedExecutionHandler);

        }

        if (this.allowCoreThreadTimeOut) {
            executor.allowCoreThreadTimeOut(true);
        }

        this.threadPoolExecutor = executor;
        return executor;
    }
```

实际上就是通过构造方法实例化 `java.util.concurrent.ThreadPoolExecutor` 对象，并设置相应参数。
