---
title: 响应式编程系列（四）Reactor 调度器总结
date: 2020-08-30 21:48:49
updated:
tags: [响应式编程, Java]
typora-root-url: ..
---

# 如何创建调度器？

通过工厂类 `Schedulers` 创建调取器： 

https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html

| Return a shared instance               | Return a new instance    | Description                                              | Notes                                                        |
| -------------------------------------- | ------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ |
| `immediate()`                          | /                        | No execution context                                     | at processing time, the submitted `Runnable` will be directly executed, effectively running them on the current `Thread` (can be seen as a "null object" or no-op `Scheduler`). |
| `single()`                             | `newSingle(...)`         | A single, reusable thread.                               | Note that this method reuses the same thread for all callers, until the Scheduler is disposed. |
| ~~`elastic()`~~                        | ~~`newElastic(...)`~~    | An unbounded elastic thread pool.                        | This one is no longer preferred with the introduction of `Schedulers.boundedElastic()`, as it has a tendency to hide backpressure problems and lead to too many threads (see below). |
| `boundedElastic()`                     | `newBoundedElastic(...)` | A bounded elastic thread pool.                           | Like its predecessor `elastic()`, it creates new worker pools as needed and reuses idle ones. Worker pools that stay idle for too long (the default is 60s) are also disposed.<br/> Unlike its predecessor `elastic()`, it has a cap on the number of backing threads it can create (**default is number of CPU cores x 10**). Up to 100 000 tasks submitted after the cap has been reached are enqueued and will be re-scheduled when a thread becomes available.<br/>**This is a better choice for I/O blocking work.** While it is made to help with legacy blocking code if it cannot be avoided. `Schedulers.boundedElastic()` is a handy way to give a blocking process its own thread so that it does not tie up other resources. See [How Do I Wrap a Synchronous, Blocking Call?](https://projectreactor.io/docs/core/release/reference/index.html#faq.wrap-blocking), but doesn’t pressure the system too much with new threads. |
| `parallel()`                           | `newParallel(...)`       | A fixed pool of workers that is tuned for parallel work. | It creates as many workers as you have CPU cores.            |
| `fromExecutorService(ExecutorService)` |                          | A Customize thread pool.                                 | Create a `Scheduler` out of any pre-existing `ExecutorService` |

delayElements
	Signals are delayed and continue on the parallel default Scheduler
	Signals are delayed and continue on an user-specified Scheduler

# 如何使用调度器？

> Reactor offers two means of switching the execution context (or `Scheduler`) in a reactive chain: `publishOn` and `subscribeOn`. Both take a `Scheduler` and let you switch the execution context to that scheduler. But the placement of `publishOn` in the chain matters, while the placement of `subscribeOn` does not. To understand that difference, you first have to remember that [nothing happens until you subscribe](https://projectreactor.io/docs/core/release/reference/index.html#reactive.subscribe).
>
> Let's have a closer look at the `publishOn` and `subscribeOn` operators:

## 例子一

演示流是运行在 `subscribe()` 方法调用的线程上，且大多数操作符继续在前一个操作符执行的线程中工作。

> most operators continue working in the `Thread` on which the previous operator executed. Unless specified, the topmost operator (the source) itself runs on the `Thread` in which the `subscribe()` call was made. The following example runs a `Mono` in a new thread:

```java
        // The Mono<String> is assembled in thread main.
        final Mono<String> mono =
                Mono.fromSupplier(() -> {
                    log.info("fromSupplier");
                    return "hello";
                })
                .map(msg -> {
                    log.info("map");
                    return msg + " world";
                });

        Thread t = new Thread(() ->
                // However, it is subscribed to in thread Thread-0.
                // As a consequence, all callbacks (fromSupplier, map, onNext) actually run in Thread-0
                mono.subscribe(log::info)
        );
        t.start();
        t.join();
```

输出结果：

```
21:02:18.436 [Thread-0] INFO FluxTest - fromSupplier
21:02:18.436 [Thread-0] INFO FluxTest - map
21:02:18.437 [Thread-0] INFO FluxTest - hello world
```

## 例子二

演示如何使用 `subscribeOn` 方法，简化上述例子一。

```java
        CountDownLatch countDownLatch = new CountDownLatch(1);

        // The Mono<String> is assembled in thread main.
        Mono.fromSupplier(() -> {
                    log.info("fromSupplier");
                    return "hello";
                })
                .map(msg -> {
                    log.info("map");
                    return msg + " world";
                })
                .doOnTerminate(countDownLatch::countDown)
                // However, the subscribeOn switches the whole sequence on a Thread picked from Scheduler.
                .subscribeOn(Schedulers.newSingle("subscribeOn"))
                .subscribe(log::info);

        countDownLatch.await();
```

输出结果：

```
21:31:52.563 [subscribeOn-1] INFO FluxTest - fromSupplier
21:31:52.563 [subscribeOn-1] INFO FluxTest - map
21:31:52.563 [subscribeOn-1] INFO FluxTest - hello world
```

## 例子三

演示 `publishOn` 如何影响其后续操作符的执行线程。

> `publishOn` takes signals from upstream and replays them downstream while executing the callback on a worker from the associated `Scheduler`. Consequently, it **affects where the subsequent operators execute** (until another `publishOn` is chained in), as follows:

```java
        CountDownLatch countDownLatch = new CountDownLatch(1);

        // 1、The Mono<String> is assembled in thread main.
        Mono.fromSupplier(() -> {
                    log.info("fromSupplier");
                    return "hello";
                })
                .map(msg -> {
                    log.info("first map");
                    return msg + " world";
                })
                // 3、The publishOn affects where the subsequent operators execute.
                .publishOn(Schedulers.newSingle("publishOn"))
                .map(msg -> {
                    log.info("second map");
                    return msg + " again";
                })
                .doOnTerminate(countDownLatch::countDown)
                // 2、However, the subscribeOn switches the whole sequence on a Thread picked from Scheduler.
                .subscribeOn(Schedulers.newSingle("subscribeOn"))
                .subscribe(log::info);

        countDownLatch.await();
```

输出结果：

```
21:32:36.975 [subscribeOn-1] INFO FluxTest - fromSupplier
21:32:36.976 [subscribeOn-1] INFO FluxTest - first map
21:32:36.977 [publishOn-2] INFO FluxTest - second map
21:32:36.977 [publishOn-2] INFO FluxTest - hello world again
```

# 如何包装同步阻塞调用？

![How Do I Wrap a Synchronous, Blocking Call?](/img/java/reactive-stream/reactor/wrap_blocking.png)

# 参考

[4.5. Threading and Schedulers](https://projectreactor.io/docs/core/release/reference/index.html#schedulers)

[8. Exposing Reactor metrics](https://projectreactor.io/docs/core/release/reference/index.html#metrics)

[Appendix C.1. How Do I Wrap a Synchronous, Blocking Call?](https://projectreactor.io/docs/core/release/reference/index.html#faq.wrap-blocking)

[Appendix C.6. How Do I Ensure Thread Affinity when I Use `publishOn()`?](https://projectreactor.io/docs/core/release/reference/index.html#faq.thread-affinity-publishon)

https://www.woolha.com/tutorials/project-reactor-publishon-vs-subscribeon-difference
