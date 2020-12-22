---
title: Java 响应式编程系列（三）Reactor Operator API 总结
date: 2020-08-25 21:48:49
updated:
tags: Java
typora-root-url: ..
---

尽管 Reactive Streams 规范并未指定任何运算符（Operators），但 Reactor 的核心价值之一就是提供了丰富的运算符。从简单的转换、过滤到复杂的编排和错误处理，涉及方方面面。

推荐通过参考文档而不是 JavaDoc 来学习 Mono/Flux API 和 Operator 操作符。参考：["which operator do I need?" appendix](https://projectreactor.io/docs/core/release/reference/docs/index.html#which-operator)

# 注意点

使用 Reactor API 时，有以下几个注意点：

一、每个 Operator API 都会返回新实例

> 在 Reactor 中，操作符（Operators）好比流水线中的工作站。 每个操作符都会向 `Publisher` 添加行为，并将上一步的 `Publisher`  包装成**新实例**。因而，整个链就被串起来，数据从第一个  `Publisher`   开始，沿着链向下移动，通过每个链接进行转换。最终，`Subscriber` 结束该流程。 注意，直到 `Subscriber` 订阅 `Publisher` 为止，什么都不会发生。
>
> 理解操作符会创建新实例这一行为，有助于避免一些常见错误，详见：[I Used an Operator on my Flux but it Doesn’t Seem to Apply. What Gives?](https://projectreactor.io/docs/core/release/reference/#faq.chain)

二、Nothing Happens Until You `subscribe()`

> Reactor 中，当您编写 `Publisher` 链时，仅用于描述异步处理的抽象过程，默认情况下数据不会开始处理。只有通过订阅，将 `Publisher` 与 `Subscriber` 绑定在一起时，才能触发整个链中的数据流处理。这是由其内部实现方式决定的：`Subscriber` 发出**请求信号**并向上传播，直到源 `Publisher`。

![org.reactivestream_api](/img/java/reactive-stream/reactive-stream/org.reactivestream_api.png)

三、使用背压（Backpressure）进行流量控制

> Propagating signals upstream is also used to implement **backpressure**, which we described in the assembly line analogy as a feedback signal sent up the line when a workstation processes more slowly than an upstream workstation.
>
> The real mechanism defined by the Reactive Streams specification is pretty close to the analogy: A subscriber can work in *unbounded* mode and let the source push all the data at its fastest achievable rate or it can use the `request` mechanism to signal the source that it is ready to process at most `n` elements.
>
> Intermediate operators can also change the request in-transit. Imagine a `buffer` operator that groups elements in batches of ten. If the subscriber requests one buffer, it is acceptable for the source to produce ten elements. Some operators also implement **prefetching** strategies, which avoid `request(1)` round-trips and is beneficial if producing the elements before they are requested is not too costly.
>
> This transforms the push model into a **push-pull hybrid**, where the downstream can pull n elements from upstream if they are readily available. But if the elements are not ready, they get pushed by the upstream whenever they are produced.

# Mono

https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html

> for [0|1] elements

![mono](/img/java/reactive-stream/reactor/mono/mono.svg)

## 创建 Mono 流

![mono_create](/img/java/reactive-stream/reactor/mono/mono_create.png)

## 中间操作

> Each operator adds behavior to a `Publisher` and wraps the previous step’s `Publisher` into a new instance. The whole chain is thus linked, such that data originates from the first `Publisher` and moves down the chain, transformed by each link. Eventually, a `Subscriber` finishes the process. Remember that nothing happens until a `Subscriber` subscribes to a `Publisher`.
>
> While the Reactive Streams specification does not specify operators at all, one of the best added values of reactive libraries, such as Reactor, **is the rich vocabulary of operators** that they provide. These cover a lot of ground, from simple transformation and filtering to complex orchestration and error handling.

### 转换操作

![mono_map](/img/java/reactive-stream/reactor/mono/mono_map.png)

### 空值处理

![mono_ifempty](/img/java/reactive-stream/reactor/mono/mono_ifempty.png)

### 执行操作

![mono_od](/img/java/reactive-stream/reactor/mono/mono_do.png)

## 异常处理

![mono_error](/img/java/reactive-stream/reactor/mono/mono_error.png)

## 终结操作

### 阻塞返回结果

![mono_block](/img/java/reactive-stream/reactor/mono/mono_block.png)

# Flux

https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html

> for [N] elements

![flux](/img/java/reactive-stream/reactor/flux/flux.svg)

## 创建 Flux 流

普通创建

![flux_create](/img/java/reactive-stream/reactor/flux/flux_create.png)

定时创建

![flux_create_interval](/img/java/reactive-stream/reactor/flux/flux_create_interval.png)

合并创建

![flux_create_combine](/img/java/reactive-stream/reactor/flux/flux_create_combine.png)

编程式创建

![flux_create_2](/img/java/reactive-stream/reactor/flux/flux_create_2.png)

其它

![flux_create_switchOnNext](/img/java/reactive-stream/reactor/flux/flux_create_switchOnNext.png)

## 中间操作

### 转换操作

Flux 转 Mono：

![flux_to_mono](/img/java/reactive-stream/reactor/flux/flux_to_mono.png)

map、flatMap：

![flux_map](/img/java/reactive-stream/reactor/flux/flux_map.png)

转成并行流：

![flux_parallel](/img/java/reactive-stream/reactor/flux/flux_parallel.png)

### 空值处理

![flux_ifempty](/img/java/reactive-stream/reactor/flux/flux_ifempty.png)

### 排序

![flux_sort](/img/java/reactive-stream/reactor/flux/flux_sort.png)

### 去重

![flux_distinct](/img/java/reactive-stream/reactor/flux/flux_distinct.png)

### 分组

![flux_window](/img/java/reactive-stream/reactor/flux/flux_window.png)

### 合并

![flux_concat](/img/java/reactive-stream/reactor/flux/flux_concat.png)

![flux_merge](/img/java/reactive-stream/reactor/flux/flux_merge.png)

### 压缩

![flux_zip](/img/java/reactive-stream/reactor/flux/flux_zip.png)

### 获取元素

例如用于获取指定个数。

![flux_take](/img/java/reactive-stream/reactor/flux/flux_take.png)

### 延迟处理

![flux_delay](/img/java/reactive-stream/reactor/flux/flux_delay.png)

### 缓冲

![flux_buffer](/img/java/reactive-stream/reactor/flux/flux_buffer.png)

### 订阅

订阅后可以使用 `Disposable` API 停止 FLux 流。

![flux_subscribe](/img/java/reactive-stream/reactor/flux/flux_subscribe.png)

### 计数

`count`

### 异常处理

![flux_error](/img/java/reactive-stream/reactor/flux/flux_error.png)

### 执行操作

![flux_do](/img/java/reactive-stream/reactor/flux/flux_do.png)

## 终结操作

### 阻塞返回结果

![flux_block](/img/java/reactive-stream/reactor/flux/flux_block.png)

# 参考

[Reactor 框架](https://projectreactor.io/)，实现 Reactive Streams 规范，并扩展大量特性

["which operator do I need?" appendix](https://projectreactor.io/docs/core/release/reference/docs/index.html#which-operator)