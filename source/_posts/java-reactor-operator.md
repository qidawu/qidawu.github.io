---
title: 响应式编程系列（三）Reactor 操作符总结
date: 2020-08-25 21:48:49
updated:
tags: [响应式编程, Java]
typora-root-url: ..
---

Reactive Streams 规范并未提供任何操作符（Operators），而 Reactor 框架的核心价值之一就是提供了丰富的操作符。从简单的转换、过滤到复杂的编排和错误处理，涉及方方面面。

推荐通过参考文档而不是 JavaDoc 来学习 Mono/Flux API 和 Operator 操作符。参考：[Appendix A: Which operator do I need?](https://projectreactor.io/docs/core/release/reference/docs/index.html#which-operator)

# 注意点

使用 Reactor API 时，有以下几个注意点：

一、每个 Operator API 都会返回新实例

> 在 Reactor 中，操作符（Operators）好比流水线中的工作站。 每个操作符都会向 `Publisher` 添加行为，并将上一步的 `Publisher`  包装成**新实例**。因而，整个链就被串起来，数据从第一个  `Publisher`   开始，沿着链向下移动，通过每个链接进行转换。最终，`Subscriber` 结束该流程。 注意，直到 `Subscriber` 订阅 `Publisher` 为止，什么都不会发生。
>
> 理解操作符会创建新实例这一行为，有助于避免一些常见错误，详见：[I Used an Operator on my Flux but it Doesn’t Seem to Apply. What Gives?](https://projectreactor.io/docs/core/release/reference/#faq.chain)

二、Nothing Happens Until You `subscribe()`

> Reactor 中，当您编写 `Publisher` 链时，仅用于描述异步处理的抽象过程，默认情况下数据不会开始处理。只有通过订阅，将 `Publisher` 与 `Subscriber` 绑定在一起时，才能触发整个链中的数据流处理。这是由其内部实现方式决定的：`Subscriber` 发出**请求信号**并向上传播，直到源 `Publisher`。

![org.reactivestream_api](/img/java/reactive-stream/reactive-stream/org.reactivestream_api.png)

详见：[C.2. I Used an Operator on my `Flux` but it Doesn’t Seem to Apply. What Gives?](https://projectreactor.io/docs/core/release/reference/index.html#faq.chain)

三、使用背压（Backpressure）进行流量控制

> Propagating signals upstream is also used to implement **backpressure**, which we described in the assembly line analogy as a feedback signal sent up the line when a workstation processes more slowly than an upstream workstation.
>
> The real mechanism defined by the Reactive Streams specification is pretty close to the analogy: A subscriber can work in *unbounded* mode and let the source push all the data at its fastest achievable rate or it can use the `request` mechanism to signal the source that it is ready to process at most `n` elements.
>
> Intermediate operators can also change the request in-transit. Imagine a `buffer` operator that groups elements in batches of ten. If the subscriber requests one buffer, it is acceptable for the source to produce ten elements. Some operators also implement **prefetching** strategies, which avoid `request(1)` round-trips and is beneficial if producing the elements before they are requested is not too costly.
>
> This transforms the push model into a **push-pull hybrid**, where the downstream can pull n elements from upstream if they are readily available. But if the elements are not ready, they get pushed by the upstream whenever they are produced.

# 我该使用哪个操作符？

## Creating a New Sequence…

### Mono

https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html

> for [0|1] elements

![mono](/img/java/reactive-stream/reactor/mono/mono.svg)

创建 Mono 流：

![mono_create](/img/java/reactive-stream/reactor/mono/mono_create.png)

### Flux

https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html

> for [N] elements

![flux](/img/java/reactive-stream/reactor/flux/flux.svg)

普通创建：

![flux_create](/img/java/reactive-stream/reactor/flux/flux_create.png)

遍历创建：



定时创建：

![flux_create_interval](/img/java/reactive-stream/reactor/flux/flux_create_interval.png)

合并创建：

![flux_create_combine](/img/java/reactive-stream/reactor/flux/flux_create_combine.png)

编程式创建：

![flux_create_2](/img/java/reactive-stream/reactor/flux/flux_create_2.png)

其它：

![flux_create_switchOnNext](/img/java/reactive-stream/reactor/flux/flux_create_switchOnNext.png)

## 中间操作

> Each operator adds behavior to a `Publisher` and wraps the previous step’s `Publisher` into a new instance. The whole chain is thus linked, such that data originates from the first `Publisher` and moves down the chain, transformed by each link. Eventually, a `Subscriber` finishes the process. Remember that nothing happens until a `Subscriber` subscribes to a `Publisher`.
>
> While the Reactive Streams specification does not specify operators at all, one of the best added values of reactive libraries, such as Reactor, **is the rich vocabulary of operators** that they provide. These cover a lot of ground, from simple transformation and filtering to complex orchestration and error handling.

### Transforming an Existing Sequence

常用的如下：

- I want to transform existing data:

  - on a 1-to-1 basis (eg. strings to their length): `map` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#map-java.util.function.Function-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#map-java.util.function.Function-))

    - …by just casting it: `cast` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#cast-java.lang.Class-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#cast-java.lang.Class-))
    - …in order to materialize each source value’s index: [Flux#index](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#index--)

  - on a 1-to-n basis (eg. strings to their characters): `flatMap` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#flatMap-java.util.function.Function-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#flatMap-java.util.function.Function-)) + use a factory method

  - running an asynchronous task for each source item (eg. urls to http request): `flatMap` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#flatMap-java.util.function.Function-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#flatMap-java.util.function.Function-)) + an async [Publisher](https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/Publisher.html?is-external=true)-returning method

    - …ignoring some data: conditionally return a [Mono.empty()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#empty--) in the flatMap lambda

    - …retaining the original sequence order: [Flux#flatMapSequential](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#flatMapSequential-java.util.function.Function-) (this triggers the async processes immediately but reorders the results)

    - …where the async task can return multiple values, from a [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) source: [Mono#flatMapMany](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#flatMapMany-java.util.function.Function-)

        ```java
        // Mono 转 Flux
        // Create a Flux that emits the items contained in the provided Iterable. A new iterator will be created for each subscriber.
        Mono#flatMapMany(Flux::fromIterable)
        ```

- I want to add pre-set elements to an existing sequence:

  - at the start: [Flux#startWith(T…)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#startWith-T...-)
  - at the end: [Flux#concatWithValues(T…)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#concatWithValues-T...-)

- I want to aggregate a [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html): (the `Flux#` prefix is assumed below)

  - into a List: [collectList](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#collectList--), [collectSortedList](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#collectSortedList--)

  - into a Map: [collectMap](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#collectMap-java.util.function.Function-), [collectMultiMap](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#collectMultimap-java.util.function.Function-)

  - into an arbitrary container: [collect](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#collect-java.util.stream.Collector-)

  - into the size of the sequence: [count](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#count--) (`Mono.empty()` 不计入 `count`)

  - by applying a function between each element (eg. running sum): [reduce](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#reduce-A-java.util.function.BiFunction-)
    
      ```java
      Flux.range(0, 5)
        .reduce(Integer::sum)  // 两两相加
        .map(Objects::toString)
        .subscribe(log::info);
      ```
      
    - …but emitting each intermediary value: [scan](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#scan-A-java.util.function.BiFunction-)
    
  - into a boolean value from a predicate:
    - applied to all values (AND): [all](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#all-java.util.function.Predicate-)
    - applied to at least one value (OR): [any](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#any-java.util.function.Predicate-)
    - testing the presence of any value: [hasElements](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#hasElements--) *(there is a [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) equivalent in [hasElement](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#hasElement--))*
    - testing the presence of a specific value: [hasElement(T)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#hasElement-T-)

- I want to combine publishers…

  - in sequential order: [Flux#concat](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#concat-org.reactivestreams.Publisher...-) or `.concatWith(other)` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#concatWith-org.reactivestreams.Publisher-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#concatWith-org.reactivestreams.Publisher-))
    - …but delaying any error until remaining publishers have been emitted: [Flux#concatDelayError](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#concatDelayError-org.reactivestreams.Publisher-)
    - …but eagerly subscribing to subsequent publishers: [Flux#mergeSequential](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#mergeSequential-int-org.reactivestreams.Publisher...-)
  - in emission order (combined items emitted as they come): [Flux#merge](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#merge-int-org.reactivestreams.Publisher...-) / `.mergeWith(other)` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#mergeWith-org.reactivestreams.Publisher-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#mergeWith-org.reactivestreams.Publisher-))
    - …with different types (transforming merge): [Flux#zip](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#zip-java.util.function.Function-org.reactivestreams.Publisher...-) / [Flux#zipWith](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#zipWith-org.reactivestreams.Publisher-)
  - by pairing values:
    - from 2 Monos into a [Tuple2](https://projectreactor.io/docs/core/release/api/reactor/util/function/Tuple2.html): [Mono#zipWith](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#zipWith-reactor.core.publisher.Mono-)
    - from n Monos when they all completed: [Mono#zip](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#zip-java.util.function.Function-reactor.core.publisher.Mono...-)
  - by coordinating their termination:
    - from 1 Mono and any source into a [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html): [Mono#and](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#and-org.reactivestreams.Publisher-)
    - from n sources when they all completed: [Mono#when](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#when-java.lang.Iterable-)
    - into an arbitrary container type:
      - each time all sides have emitted: [Flux#zip](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#zip-java.util.function.Function-org.reactivestreams.Publisher...-) (up to the smallest cardinality)
      - each time a new value arrives at either side: [Flux#combineLatest](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#combineLatest-java.util.function.Function-int-org.reactivestreams.Publisher...-)
  - selecting the first publisher which…
    - produces a *value* (`onNext`): `firstWithValue` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#firstWithValue-java.lang.Iterable-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#firstWithValue-java.lang.Iterable-))
    - produces *any signal*: `firstWithSignal` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#firstWithSignal-java.lang.Iterable-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#firstWithSignal-java.lang.Iterable-))
  - triggered by the elements in a source sequence: [switchMap](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#switchMap-java.util.function.Function-) (each source element is mapped to a Publisher)
  - triggered by the start of the next publisher in a sequence of publishers: [switchOnNext](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#switchOnNext-org.reactivestreams.Publisher-)

- I want to repeat an existing sequence: `repeat` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#repeat--)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#repeat--))

  - ...

- I have an empty sequence but…

  - I want a value instead: `defaultIfEmpty` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#defaultIfEmpty-T-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#defaultIfEmpty-T-))

  - I want another sequence instead: `switchIfEmpty` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#switchIfEmpty-org.reactivestreams.Publisher-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#switchIfEmpty-reactor.core.publisher.Mono-))

      ```java
      Flux.just(0, 1, 2, 3)
        .filter(i -> i < 0)
        .next()
        .doOnNext(i -> log.info("Exist a item: {}", i))
        .switchIfEmpty(Mono.just(-1))
        .map(Objects::toString)
        .subscribe(log::info);
      ```

- I have a sequence but I am not interested in values: `ignoreElements` ([Flux.ignoreElements()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#ignoreElements--)|[Mono.ignoreElement()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#ignoreElement--))

  - …and I want the completion represented as a [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html): `then` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#then--)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#then--))
  - …and I want to wait for another task to complete at the end: `thenEmpty` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#thenEmpty-org.reactivestreams.Publisher-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#thenEmpty-org.reactivestreams.Publisher-))
  - …and I want to switch to another [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) at the end: [Mono#then(mono)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#then-reactor.core.publisher.Mono-)
  - …and I want to emit a single value at the end: [Mono#thenReturn(T)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#thenReturn-V-)
  - …and I want to switch to a [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) at the end: `thenMany` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#thenMany-org.reactivestreams.Publisher-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#thenMany-org.reactivestreams.Publisher-))

- ...

### Filtering a Sequence

- I want to filter a sequence:
  - based on an arbitrary criteria: `filter` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#filter-java.util.function.Predicate-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#filter-java.util.function.Predicate-))
    - …that is asynchronously computed: `filterWhen` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#filterWhen-java.util.function.Function-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#filterWhen-java.util.function.Function-))
  - restricting on the type of the emitted objects: `ofType` ([Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#ofType-java.lang.Class-)|[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#ofType-java.lang.Class-))
  - by ignoring the values altogether: `ignoreElements` ([Flux.ignoreElements()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#ignoreElements--)|[Mono.ignoreElement()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#ignoreElement--))
  - by ignoring duplicates:
    - in the whole sequence (logical set): [Flux#distinct](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#distinct--)
    - between subsequently emitted items (deduplication): [Flux#distinctUntilChanged](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#distinctUntilChanged--)
- I want to keep only a subset of the sequence:
  - by taking N elements:
    - at the beginning of the sequence: [Flux#take(long, true)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#take-long-boolean-)
      - …requesting an unbounded amount from upstream: [Flux#take(long, false)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#take-long-boolean-)
      - …based on a duration: [Flux#take(Duration)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#take-java.time.Duration-)
      - …only the first element, as a [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html): [Flux#next()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#next--)
      - …using [request(N)](https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/Subscription.html#request(long)) rather than cancellation: [Flux#limitRequest(long)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#limitRequest-long-)
    - at the end of the sequence: [Flux#takeLast](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#takeLast-int-)
    - until a criteria is met (inclusive): [Flux#takeUntil](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#takeUntil-java.util.function.Predicate-) (predicate-based), [Flux#takeUntilOther](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#takeUntilOther-org.reactivestreams.Publisher-) (companion publisher-based)
    - while a criteria is met (exclusive): [Flux#takeWhile](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#takeWhile-java.util.function.Predicate-)
  - by taking at most 1 element:
    - at a specific position: [Flux#elementAt](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#elementAt-int-)
    - at the end: [.takeLast(1)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#takeLast-int-)
      - …and emit an error if empty: [Flux#last()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#last--)
      - …and emit a default value if empty: [Flux#last(T)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#last-T-)
  - by skipping elements:
    - at the beginning of the sequence: [Flux#skip(long)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#skip-long-)
      - …based on a duration: [Flux#skip(Duration)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#skip-java.time.Duration-)
    - at the end of the sequence: [Flux#skipLast](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#skipLast-int-)
    - until a criteria is met (inclusive): [Flux#skipUntil](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#skipUntil-java.util.function.Predicate-) (predicate-based), [Flux#skipUntilOther](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#skipUntilOther-org.reactivestreams.Publisher-) (companion publisher-based)
    - while a criteria is met (exclusive): [Flux#skipWhile](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#skipWhile-java.util.function.Predicate-)
  - by sampling items:
    - by duration: [Flux#sample(Duration)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#sample-java.time.Duration-)
      - but keeping the first element in the sampling window instead of the last: [sampleFirst](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#sampleFirst-java.time.Duration-)
    - by a publisher-based window: [Flux#sample(Publisher)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#sample-org.reactivestreams.Publisher-)
    - based on a publisher "timing out": [Flux#sampleTimeout](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#sampleTimeout-java.util.function.Function-) (each element triggers a publisher, and is emitted if that publisher does not overlap with the next)
- I expect at most 1 element (error if more than one)…
  - and I want an error if the sequence is empty: [Flux#single()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#single--)
  - and I want a default value if the sequence is empty: [Flux#single(T)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#single-T-)
  - and I accept an empty sequence as well: [Flux#singleOrEmpty](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#singleOrEmpty--)

| 方法                   | 注释                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `take(long n)`         | Take only the first N values from this `Flux`, if available. |
| `takeLast(long n)`     | Emit the last N values this `Flux` emitted before its completion. |
| `last()`               | Emit the last element observed before complete signal as a `Mono`, or emit `NoSuchElementException` error if the source was empty. |
| `last(T defaultValue)` | Emit the last element observed before complete signal as a `Mono`, or emit the **defaultValue** if the source was empty. |

### Peeking into a Sequence

![Reactive callback](/img/java/reactive-stream/reactive-stream/process_of_reactive_stream_2.png)

| 方法                        | 入参                                                         | 注释                                                         |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `doOnSubscribe`             | `Consumer<? super Subscription>`                             | Add behavior triggered when the `Mono` is subscribed.        |
| `doOnCancel`                | `Runnable`                                                   | Add behavior triggered when the `Mono` is cancelled.         |
| `doOnRequest`               | `LongConsumer`                                               | Add behavior triggering a `LongConsumer` when the Mono receives any request. |
| `doOnNext`                  | `Consumer<? super T>`                                        | Add behavior triggered when the `Mono` emits a data successfully. |
| do on Complete ...          |                                                              |                                                              |
| `doOnComplete`              | `Runnable`                                                   | Add behavior triggered when the `Flux` completes successfully. |
| `doOnSuccess`               | `Consumer<? super T>`                                        | Add behavior triggered when the `Mono` completes successfully.<br/>* `null` : completed without data<br/>* `T`: completed with data |
| `doOnError`                 | `Consumer<? super Throwable>`<br/>`Class<E>, Consumer<? super E>`<br/>`Predicate<? super Throwable>, Consumer<? super Throwable>` | Add behavior triggered when the `Mono` completes with an error. |
| `doOnTerminate`             | `Runnable`                                                   | completion or error                                          |
| `doAfterTerminate`          | `Runnable`                                                   | completion or error but **after** it has been propagated downstream |
| `doFinally`                 | `Consumer<SignalType>`                                       | any terminating condition (complete, error, cancel).<br/>    |
| ~~`doOnSuccessOrError`~~    |                                                              | **Deprecated**, will be removed in 3.5.0. Prefer using [`doOnNext(Consumer)`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#doOnNext-java.util.function.Consumer-), [`doOnError(Consumer)`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#doOnError-java.util.function.Consumer-), [`doOnTerminate(Runnable)`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#doOnTerminate-java.lang.Runnable-) or [`doOnSuccess(Consumer)`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#doOnSuccess-java.util.function.Consumer-). <br/>Add behavior triggered when the `Mono` terminates, either by completing successfully or with an error.<br/>* `nul`l, `null` : completing without data<br/>* `T`, `null` : completing with data<br/>* `null`, `Throwable` : failing with/without data |
| ~~`doAfterSuccessOrError`~~ |                                                              | **Deprecated**, will be removed in 3.5.0. Prefer using [`doAfterTerminate(Runnable)`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#doAfterTerminate-java.lang.Runnable-) or [`doFinally(Consumer)`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#doFinally-java.util.function.Consumer-).<br/><br/>Add behavior triggered after the `Mono` terminates, either by completing downstream successfully or with an error. The arguments will be null depending on success, success with data and error:<br/>* `null`, `null` : completed without data<br/>* `T`, `null` : completed with data<br/>* `null`, `Throwable` : failed with/without data |
| all events ...              |                                                              |                                                              |
| `doOnEach`                  | `Consumer<? super Signal<T>>`                                | I want to know of all events each represented as [Signal](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Signal.html) object in a callback outside the sequence: `doOnEach` |

调试类：

| 方法        | 注释                                                         |
| ----------- | ------------------------------------------------------------ |
| `log`       | Observe all Reactive Streams signals and trace them using `Logger` support. Default will use `Level.INFO` and java.util.logging. If **SLF4J** is available, it will be used instead. |
| `timestamp` | If this `Mono` is valued, emit a `Tuple2` pair of T1 the current clock time in millis (as a `Long` measured by the parallel Scheduler) and T2 the emitted data (as a T). |
| `elapsed`   |                                                              |

### Handling Errors

对于异常处理，Reactor 除了默认的立刻抛出异常的处理方式之外，还提供三类处理方式：

* 简单记录日志（`doOnError`）
* recover from errors **by falling back** (`onErrorReturn`、`onErrorResume`)
* recover from errors **by retrying** (`retry`、`retryWhen`)

| 方法                        | 注释                                                         | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `error`                     | Create a `Mono` that terminates with the specified error immediately after being subscribed to. | 创建异常流。                                                 |
| `onErrorMap`                | Transform any error emitted by this `Mono` by synchronously applying a function to it.<br/>Transform an error emitted by this `Mono` by synchronously applying a function to it <u>if the error matches the given type.</u> Otherwise let the error pass through.<br/>Transform an error emitted by this `Mono` by synchronously applying a function to it <u>if the error matches the given predicate.</u> Otherwise let the error pass through. | catching an exception and wrapping and re-throwing           |
| `onErrorReturn`             | Simply emit a captured fallback value when <u>any error</u> is observed on this `Mono`.<br/>Simply emit a captured fallback value when <u>an error of the specified type</u> is observed on this `Mono`.<br/>Simply emit a captured fallback value when <u>an error matching the given predicate</u> is observed on this `Mono`. | catching an exception and falling back to a default value    |
| `onErrorResume`             | Subscribe to a fallback publisher when <u>any error</u> occurs, using a function to choose the fallback depending on the error.<br/>Subscribe to a fallback publisher when <u>an error matching the given type</u> occurs.<br/>Subscribe to a fallback publisher when <u>an error matching a given predicate</u> occurs. | catching an exception and falling back to another `Mono`     |
| `onErrorContinue`           | Let compatible operators **upstream** recover from errors by dropping the incriminating element from the sequence and continuing with subsequent elements. | https://devdojo.com/ketonemaniac/reactor-onerrorcontinue-vs-onerrorresume |
| `retry()`<br/>`retry(long)` | Re-subscribes to this `Mono` sequence if it signals any error, indefinitely.<br/>Re-subscribes to this `Mono` sequence if it signals any error, for a fixed number of times. | retrying with a simple policy (max number of attempts)       |
| `retryWhen`                 |                                                              |                                                              |

`Mono`

![mono_error](/img/java/reactive-stream/reactor/mono/mono_error.png)

`Flux`

![flux_error](/img/java/reactive-stream/reactor/flux/flux_error.png)

受检异常处理：

> 非受检异常会被 Reactor 传播，而受检异常必须被用户代码 try catch ，为了让受检异常被 Reactor 的异常传播机制和异常处理机制支持，可以使用如下步骤处理：
>
> 1.  try catch 之后，使用 `Exceptions.propagate` 将受检异常包装为非受检异常并重新抛出传播出去。
> 2. `onError` 回调等异常处理操作获取到异常之后，可以调用 `Exceptions.unwrap` 取得原受检异常。

参考：https://projectreactor.io/docs/core/release/reference/index.html#error.handling

### Sorting a Flux

![flux_sort](/img/java/reactive-stream/reactor/flux/flux_sort.png)

### Splitting a Flux

https://projectreactor.io/docs/core/release/reference/index.html#advanced-three-sorts-batching

### 延迟处理

![flux_delay](/img/java/reactive-stream/reactor/flux/flux_delay.png)

### 转成并行流

![flux_parallel](/img/java/reactive-stream/reactor/flux/flux_parallel.png)

## 终结操作

### Subscribe a Sequence

`Mono`、`Flux`

![mono_subscribe](/img/java/reactive-stream/reactor/mono/mono_subscribe.png)

订阅后可以使用 `Disposable` API 停止 FLux 流。

### Going Back to the Synchronous World

Note: all of these methods except [Mono#toFuture](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#toFuture--) will throw an [UnsupportedOperatorException](https://docs.oracle.com/javase/8/docs/api/java/lang/UnsupportedOperationException.html?is-external=true) if called from within a [Scheduler](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Scheduler.html) marked as "non-blocking only" (by default [parallel()](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html#parallel--) and [single()](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html#single--)).

- I have a [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) and I want to:
  - block until I can get the first element: [Flux#blockFirst](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#blockFirst--)
    - …with a timeout: [Flux#blockFirst(Duration)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#blockFirst-java.time.Duration-)
  - block until I can get the last element (or null if empty): [Flux#blockLast](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#blockLast--)
    - …with a timeout: [Flux#blockLast(Duration)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#blockLast-java.time.Duration-)
  - synchronously switch to an [Iterable](https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html?is-external=true): [Flux#toIterable](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#toIterable--)
  - synchronously switch to a Java 8 [Stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html): [Flux#toStream](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#toStream--)
- I have a [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) and I want:
  - to block until I can get the value: [Mono#block](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#block--)
    - …with a timeout: [Mono#block(Duration)](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#block-java.time.Duration-)
  - a [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html): [Mono#toFuture](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#toFuture--)

# 参考

[Reactor 框架](https://projectreactor.io/)，实现 Reactive Streams 规范，并扩展大量特性

[Appendix A: Which operator do I need?](https://projectreactor.io/docs/core/release/reference/docs/index.html#which-operator)

https://zhuanlan.zhihu.com/p/35964846

