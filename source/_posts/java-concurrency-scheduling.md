---
title: 并发编程系列（四）定时任务调度类总结
date: 2020-05-05 15:07:11
updated: 
tags: [并发编程, Java]
typora-root-url: ..
---

# `Timer` & `TimerTask`

`java.util.Timer` 是 JDK 中提供的一个定时器工具类，使用的时候会在主线程之外起一个单独的线程执行指定的定时任务 `java.util.TimerTask`，可以指定执行一次或者反复执行多次。

`java.util.TimerTask` 是一个实现了 `java.lang.Runnable` 接口的抽象类，代表一个可以被 `Timer` 执行的任务。

![TimerTask](/img/java/concurrent/TimerTask.png)

用法如下（参考 ZooKeeper 源码）：

![TimerTask usage](/img/java/concurrent/TimerTask_usage.png)

# `ScheduledExecutorService`

⚠️ 注意：尽量别用 `java.util.TimerTask`，如要用一定要捕获处理好异常，一般建议使用 `java.util.concurrent.ScheduledExecutorService` 代替：

![Executor](/img/java/concurrent/subtypes_of_Executor.png)

参考《阿里巴巴 Java 开发手册》：

![TimerTask problem](/img/java/concurrent/TimerTask_problem.png)

用法如下（参考 Ribbon 源码 [`com.netflix.loadbalancer.PollingServerListUpdater`](https://github.com/Netflix/ribbon/blob/master/ribbon-loadbalancer/src/main/java/com/netflix/loadbalancer/PollingServerListUpdater.java)）：

```java
        ScheduledThreadPoolExecutor _serverListRefreshExecutor = Executors.newScheduledThreadPool(POOL_SIZE, new ThreadFactoryBuilder()
                    .setNameFormat("PollingServerListUpdater-%d")
                    .setDaemon(true)
                    .build()
        );
        _serverListRefreshExecutor.scheduleWithFixedDelay(
            wrapperRunnable,
            initialDelayMs,
            refreshIntervalMs,
            TimeUnit.MILLISECONDS
        );
```

# 参考

《[JDK 中的 Timer 和 TimerTask 详解](https://www.cnblogs.com/lingiu/p/3782813.html)》

《[面试官：知道时间轮算法吗？在 Netty 和 Kafka 中如何应用的？](https://mp.weixin.qq.com/s/wcLndaUVwWJ-Sh9nlMregw)》

