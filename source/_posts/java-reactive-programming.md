---
title: 响应式编程系列（一）Java 响应式编程总结
date: 2020-08-01 21:48:49
updated:
tags: [响应式编程, Java]
typora-root-url: ..
---

# 历史

响应式编程（Reactive Programming）概念最早于上世纪九十年代被提出，微软为 .NET 生态开发了 Reactive Extensions (Rx) 库用于支持响应式编程，后来 Netflix 开发了 RxJava，为 JVM 生态实现了响应式编程。随着时间的推移，2015 年 [Reactive Stream](https://www.reactive-streams.org/)（响应式流）规范诞生，为 JVM 上的响应式编程定义了一组接口和交互规则。[RxJava](https://github.com/ReactiveX/RxJava) 从 RxJava 2 开始实现 Reactive Stream 规范。同时 MongoDB、[Reactor](https://projectreactor.io/)、Slick 等也相继实现了 Reactive Stream 规范。

Spring Framework 5 推出了[响应式 Web 框架](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)。

Java 9 引入了[响应式编程的 API](http://openjdk.java.net/jeps/266)，将 Reactive Stream 规范定义的四个接口集成到了 [`java.util.concurrent.Flow`](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html) 类中。Java 9 提供了 `SubmissionPublisher` 和 `ConsumerSubscriber` 两个默认实现。

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

# 定义

响应式编程（Reactive Programing）是一种基于数据流（data stream）和变化传递（propagation of change）的声明式（declarative）的编程范式。

# 设计思想

响应式编程范式通常在面向对象语言中作为**观察者模式**的扩展出现。可以将其与大家熟知的**迭代器模式**作对比，主要区别在于：

|          | 迭代器（Iterator） | 响应式流（Reactive Stream） |
| -------- | ------------------ | --------------------------- |
| 设计模式 | 迭代器模式         | 观察者模式                  |
| 数据方向 | 拉模式（PULL）     | 推模式（PUSH）              |
| 获取数据 | `T next()`         | `onNext(T)`                 |
| 处理完成 | `hasNext()`        | `onCompleted()`             |
| 异常处理 | `throws Exception` | `onError(Exception)`        |

Java 8 引入了 Stream 用于流的操作，Java 9 引入的 Flow 也是数据流的操作。相比之下：

* Stream 更侧重于流的过滤、映射、整合、收集，使用的是 **PULL** 模式。
* 而 Flow/RxJava/Reactor 更侧重于流的产生与消费，使用的是 **PUSH** 模式 。

## 迭代器模式

参考《[Iterator API 总结](/2018/05/01/java-collections-iterator/)》

## 观察者模式

> 观察者模式是一种行为型设计模式，允许你定义一种订阅机制，可在对象事件发生时**主动通知**多个 “观察” 该对象的其它对象。

![Observer](/img/java/design-pattern/Observer.png)

在响应式流中，上述操作由 `Publisher-Subscriber` 负责。由 `Publisher` 生产新值并推送给 `Subscriber`，这个“推送”就是响应式的关键，亦即“**变化传递（propagation of change）**”。另外，应用于被推送值的操作（Operator）是“**声明式**”而不是“命令式”的：开发者表达的是计算逻辑，而不是描述其具体的控制流程。

流程如下：

```
onNext x 0..N [onError | onComplete]
```

# Reactive Streams 规范

## 依赖

Reactive Stream（响应式流）规范的 Maven 依赖如下：

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

Reactive Streams 规范

* https://www.reactive-streams.org/
* https://en.wikipedia.org/wiki/Reactive_Streams
* https://zhuanlan.zhihu.com/p/41342507

ReactiveX 系列

* http://reactivex.io/
* https://github.com/ReactiveX
  * RxJava、RxKotlin、RxSwift、RxAndroid、RxJS、RxPY、RxGo、…
    * 《Reactive Programming with RxJava》
    * 《[如何选择操作符？ - RxSwift](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree.html)》

Reactor 框架

* https://projectreactor.io/
* 实现 Reactive Streams 规范，并扩展大量特性

Spring

* https://spring.io/reactive
* [Web on Reactive Stack - Spring](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)
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

http://openjdk.java.net/jeps/266
