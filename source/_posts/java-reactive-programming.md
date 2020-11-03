---
title: Java 响应式编程系列（一）响应式编程总结
date: 2020-08-01 21:48:49
updated:
tags: Java
typora-root-url: ..
---

# 响应式编程是什么

一种基于数据流（data stream）和变化传递（propagation of change）的声明式（declarative）的编程范式。

# 响应式编程历史

响应式编程（Reactive Programming）概念最早于上世纪九十年代被提出，微软为 .NET 生态开发了 Reactive Extensions (Rx) 库用于支持响应式编程，后来 Netflix 开发了 RxJava，为 JVM 生态实现了响应式编程。随着时间的推移，2015 年 [Reactive Stream](https://www.reactive-streams.org/)（响应式流）规范诞生，为 JVM 上的响应式编程定义了一组接口和交互规则。[RxJava](https://github.com/ReactiveX/RxJava) 从 RxJava 2 开始实现 Reactive Stream 规范。同时 MongoDB、[Reactor](https://projectreactor.io/)、Slick 等也相继实现了 Reactive Stream 规范。

Spring Framework 5 推出了[响应式 Web 框架](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)。

Java 9 引入了[响应式编程的 API](http://openjdk.java.net/jeps/266)，将 Reactive Stream 规范定义的四个接口集成到了 [`java.util.concurrent.Flow`](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html) 类中。Java 9 提供了 `SubmissionPublisher` 和 `ConsumerSubscriber 两个默认实现。

Java 8 引入了 Stream 用于流的操作，Java 9 引入的 Flow 也是数据流的操作。相比之下，Stream 更侧重于流的过滤、映射、整合、收集，使用的是 **PULL** 模式。而 Flow/RxJava/Reactor 更侧重于流的产生与消费，使用的是 **PUSH** 模式 。

```
+--------------------------+     +-------------+     +------------------+     +-------------------------------+
| Reactive Extensions (Rx) |     | RxJava 1.x  |     | Reactive Streams |     | RxJava 2                      |
| by Microsoft             +-----> by Netflix  +-----> Specification    +-----> (Supporting Reactive Streams) |
| for .NET                 |     | for Java 6+ |     | for JVM          |     | for Java 6+                   |
+--------------------------+     +-------------+     +------------------+     +-------------------------------+
                                                                                              |
                                                                                              |
                                                                                              |
                     +-----------------------+     +--------------------+     +---------------v---------------+
                     | Java 9 Standard       |     | Spring Framework 5 |     | Project Reactor              |
                     | (JEP-266 by Doug Lea) <-----+ Reactive Stack     <-----+ (Supporting Reactive Streams) |
                     |                       |     |                    |     | for Java 8+                   |
                     +-----------------------+     +--------------------+     +-------------------------------+

```

# 响应式编程介绍

响应式编程范式通常在面向对象语言中作为**观察者模式**的扩展出现。可以将其与大家熟知的**迭代器模式**作对比，主要区别在于：迭代器基于**拉模式**，而响应式流基于**推模式**。

此外，使用迭代器是一种**命令式编程范式**，即使访问值的方法仅由 `Iterable-Iterator` 负责。但实际上，是由开发者来决定何时访问序列中的 `next()` 项。

![]()

而在响应式流中，上述操作由 `Publisher-Subscriber` 负责。由 `Publisher` 生产新值并推送给 `Subscriber`，这个“推送”就是响应式的关键。另外，应用于被推送值的操作（Operator）是“**声明式**”而不是“命令式”的：开发者表达的是计算逻辑，而不是描述其具体的控制流程。

![]()

流程如下：

```
onNext x 0..N [onError | onComplete]
```

# 响应式编程优点

## 解决阻塞带来的性能浪费

现代的应用程序通常有大量并发请求。即使现代的硬件性能不断提高，软件性能仍然是关键瓶颈。

广义上讲，有两种方法可以提高程序的性能：

* 利用并行（parallel）使用更多 CPU 线程和更多硬件资源。
* 提升现有资源的利用率。

通常，Java 开发者使用阻塞方式来编写程序。除非达到性能瓶颈，否则这种做法可行。之后，通过增加线程数，运行类似的阻塞代码。 但这种方式很快就会导致资源争用和并发问题。

更糟糕的是，阻塞会浪费资源。试想一下，程序一旦遇到一些延迟（特别是 I/O 操作，例如数据库请求或网络请求），就会导致资源浪费，因为大量线程处于空闲状态，等待数据，甚至导致资源耗尽。使用池化技术可以提升资源利用率、避免资源耗尽，但只能缓解而不能解决根本问题。

因此，并行技术并非银弹。

## 解决传统的异步编程带来的缺点

Java 提供了两种异步编程模型：

* Callbacks: 异步方法**没有返回值**，但是带有一个额外的 `callback` 回调参数（值为 Lambda 表达式或匿名类），该参数在结果可用时被调用。
* Futures: 异步方法调用后立即返回一个 `Future<T>` 对象。异步方法负责计算出结果值 `T`，由 `Future` 对象包装起来。该结果值并非立即可用，可以轮询该 `Future` 对象，直到结果值可用为止。例如 `ExecutorService` 提供的方法 `<T> Future<T> submit(Callable<T> task)`。

两种模型都有一个共同的缺点：难以组合代码，从而导致代码可读性差、难以维护。例如，当业务逻辑复杂，步骤存在依赖关系时，会导致回调**嵌套过深**，从而导致著名的 Callback Hell 问题。

详见代码例子。

## 从命令式编程到响应式编程

![from_imperative_to_reactive_programming](/img/java/reactive-stream/reactor/from_imperative_to_reactive_programming.png)

# 响应式流规范

## 依赖

Reactive Stream 规范的 Maven 依赖如下：

```XML
<!-- https://mvnrepository.com/artifact/org.reactivestreams/reactive-streams -->
<dependency>
    <groupId>org.reactivestreams</groupId>
    <artifactId>reactive-streams</artifactId>
    <version>1.0.2</version>
</dependency>
```

## 核心接口

整个依赖包中，仅仅定义了四个核心接口：

* `org.reactivestreams.Subscription` 接口定义了连接发布者和订阅者的方法；
* `org.reactivestreams.Publisher<T>` 接口定义了发布者的方法；
* `org.reactivestreams.Subscriber<T>` 接口定义了订阅者的方法；
* `org.reactivestreams.Processor<T,R>` 接口定义了处理器；

![org.reactivestreams](/img/java/reactive-stream/reactive-stream/org.reactivestreams.png)

## 接口交互流程

简要交互如下：

![process_ofreactive_stream](/img/java/reactive-stream/reactive-stream/process_of_reactive_stream.jpg)

API 交互如下：

![process_of_reactive_stream_2](/img/java/reactive-stream/reactive-stream/process_of_reactive_stream_2.png)

# 参考

《Reactive Java Programming》

[Reactive Streams 规范](https://www.reactive-streams.org/)

https://en.wikipedia.org/wiki/Reactive_Streams

http://openjdk.java.net/jeps/266

[Reactor 框架](https://projectreactor.io/)，实现 Reactive Streams 规范，并扩展大量特性

http://reactivex.io/

* 《Reactive Programming with RxJava》



https://spring.io/reactive

[Web on Reactive Stack - Spring](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)

* 基于 Reactor 框架实现
* 默认基于 Netty 作为应用服务器
* 好处：能够以固定的线程来处理高并发（充分发挥机器的性能）
* 提供 API：
  * Spring WebFlux
  * WebClient
  * WebSockets
  * Testing
  * RSocket
  * Reactive Libraries

