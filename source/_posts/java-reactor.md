---
title: Java 响应式编程系列（二）Reactor API 总结
date: 2020-08-01 21:48:49
updated:
tags: Java
typora-root-url: ..
---

# 历史

> 响应式编程概念最早于上世纪九十年代被提出，.NET 开发了 Reactive eXtension 库来支持响应式编程，后来 Netflix 开发了 RxJava 。2015 年 Reactive Stream（响应式流）规范诞生，RxJava 从 RxJava 2 开始实现 Reactive Stream 规范。同时 MongoDB、Reactor、Slick 等也相继实现了 Reactive Stream 规范。
>
> Spring Framework 5 推出了[响应式 Web 框架](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)。
>
> Java 9 引入了响应式编程的 API。Java 9 [java.util.concurrent.Flow](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html) 类内嵌的四个接口实际上就是拷贝了 Reactive Stream 的四个接口定义。Java 9 提供了 `SubmissionPublisher` 和 `ConsumerSubscriber 两个默认实现。
>
> Java 8 引入了 Stream 用于流的操作，Java 9 引入的 Flow 也是数据流的操作。相比之下，Stream 更侧重于流的过滤、映射、整合、收集，使用的是 **PULL** 模式。而 Flow/RxJava/Reactor 更侧重于流的产生与消费，使用的是 **PUSH** 模式 。

# 总览

## Reactive Stream

Reactive Stream 规范的 Maven 依赖如下：

```XML
<!-- https://mvnrepository.com/artifact/org.reactivestreams/reactive-streams -->
<dependency>
    <groupId>org.reactivestreams</groupId>
    <artifactId>reactive-streams</artifactId>
    <version>1.0.2</version>
</dependency>
```

整个依赖包中，仅仅定义了四个核心接口：

* `org.reactivestreams.Subscription` 接口定义了连接发布者和订阅者的方法；
* `org.reactivestreams.Publisher<T>` 接口定义了发布者的方法；
* `org.reactivestreams.Subscriber<T>` 接口定义了订阅者的方法；
* `org.reactivestreams.Processor<T,R>` 接口定义了处理器；

![org.reactivestreams](/img/java/reactive/reactor/org.reactivestreams.png)

接口交互如下：

![process_ofreactive_stream](/img/java/reactive/reactor/process_ofreactive_stream.jpg)

## Reactor

Reactor 框架实现了 Reactive Stream 规范，其依赖如下：

```XML
    <dependencies>
        <!-- Reactor -->
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.projectreactor</groupId>
                <artifactId>reactor-bom</artifactId>
                <version>Californium-SR5</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

从下图可见，Reactor 框架传递依赖于 Reactive Stream：

```XML
    <dependency>
      <groupId>org.reactivestreams</groupId>
      <artifactId>reactive-streams</artifactId>
      <version>1.0.2</version>
      <scope>compile</scope>
    </dependency>
```

![reactor_dependencies](/img/java/reactive/reactor/reactor_dependencies.png)

Reactor 框架提供如下核心实现类：

`Publisher` 实现类：

![Publisher](/img/java/reactive/reactor/Publisher.png)

`Subscriber` 实现类（还有其它子类不一一例举）：

![Subscriber](/img/java/reactive/reactor/Subscriber.png)

# Mono

> for [0|1] elements

## 创建 Mono 流

![mono_create](/img/java/reactive/reactor/mono/mono_create.png)

## 中间操作

### 转换操作

![mono_map](/img/java/reactive/reactor/mono/mono_map.png)

### 空值处理

![mono_ifempty](/img/java/reactive/reactor/mono/mono_ifempty.png)

### 异常处理

![mono_error](/img/java/reactive/reactor/mono/mono_error.png)

## 终结操作

### 阻塞返回结果

![mono_block](/img/java/reactive/reactor/mono/mono_block.png)

# Flux

> for [N] elements

## 创建 Flux 流

普通创建

![flux_create](/img/java/reactive/reactor/flux/flux_create.png)

定时创建

![flux_create_interval](/img/java/reactive/reactor/flux/flux_create_interval.png)

合并创建

![flux_create_combine](/img/java/reactive/reactor/flux/flux_create_combine.png)

编程式创建

![flux_create_2](/img/java/reactive/reactor/flux/flux_create_2.png)

其它

![flux_create_switchOnNext](/img/java/reactive/reactor/flux/flux_create_switchOnNext.png)

## 中间操作

### 转换操作

Flux 转 Mono：

![flux_to_mono](/img/java/reactive/reactor/flux/flux_to_mono.png)

map、flatMap：

![flux_map](/img/java/reactive/reactor/flux/flux_map.png)

转成并行流：

![flux_parallel](/img/java/reactive/reactor/flux/flux_parallel.png)

### 排序

![flux_sort](/img/java/reactive/reactor/flux/flux_sort.png)

### 去重

![flux_distinct](/img/java/reactive/reactor/flux/flux_distinct.png)

### 分组

![flux_window](/img/java/reactive/reactor/flux/flux_window.png)

### 合并

![flux_concat](/img/java/reactive/reactor/flux/flux_concat.png)

![flux_merge](/img/java/reactive/reactor/flux/flux_merge.png)

### 压缩

![flux_zip](/img/java/reactive/reactor/flux/flux_zip.png)

### 获取元素

例如用于获取指定个数。

![flux_take](/img/java/reactive/reactor/flux/flux_take.png)

### 延迟处理

![flux_delay](/img/java/reactive/reactor/flux/flux_delay.png)

### 缓冲

![flux_buffer](/img/java/reactive/reactor/flux/flux_buffer.png)

### 订阅

订阅后可以使用 `Disposable` API 停止 FLux 流。

![flux_subscribe](/img/java/reactive/reactor/flux/flux_subscribe.png)

### 计数

`count`

### 异常处理

![flux_error](/img/java/reactive/reactor/flux/flux_error.png)

## 终结操作

### 阻塞返回结果

![flux_block](/img/java/reactive/reactor/flux/flux_block.png)

# 参考

Reactive Programming（响应式编程）

[Reactive Streams 规范](https://www.reactive-streams.org/)

[Reactor 框架](https://projectreactor.io/)，实现 Reactive Streams 规范，并大量扩展特性

《Reactive Java Programming》

《Reactive Programming with RxJava》



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

