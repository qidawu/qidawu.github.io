---
title: Spring 依赖注入总结
date: 2017-05-29 22:30:34
updated:
tags: Java
---

# 依赖注入

按照传统的做法，每个对象负责管理与自己相互协作的对象（即它所依赖的对象）的引用，这将会导致高度耦合和难以测试的代码：

```
             +---+
   +---new--->Bar|
   |         +---+
 +-+-+
 |Foo|
 +-+-+
   |         +---+
   +---new--->Baz|
             +---+
```

通过依赖注入，对象的依赖关系将由系统中负责协调各对象的**第三方组件**在创建对象的时候进行设定。对象无需自行创建或管理它们的依赖关系，依赖关系将被自动注入到需要它们的对象当中去，即做到“控制反转（IoC）”：

```
             +---+
  +--inject--+Bar|
  |          +---+
+-v-+
|Foo|
+-^-+
  |          +---+
  +--inject--+Baz|
             +---+
```

如果一个对象只通过接口（而不是具体实现或初始化过程）来表明依赖关系，那么这种依赖就能够在对象本身毫不知情的情况下，用不同的具体实现进行替换。这就是依赖注入所带来的最大收益——松耦合。

对依赖进行替换的一个最常用方法就是在测试的时候使用 mock 实现。

# 装配 Bean

创建应用对象之间协作关系的行为通常称为装配（wiring），这也是依赖注入（DI）的本质。

只有 Spring 通过它的配置，能够了解这些组成部分是如何装配起来的。这样的话，就可以在不改变所依赖的类的情况下，修改依赖关系。

Spring 提供几种配置方式，用于 bean 的声明及装配，详见《[Spring Bean 几种配置方式总结](/2017/06/04/spring-bean-wiring/)》。

# 使用场景

## Aware 接口解析

在日常的开发中，我们经常需要用到 Spring 容器本身的功能资源，可以通过 Spring 提供的一系列 `Aware` 子接口来实现具体的功能：

![Aware 接口](/img/spring/aware_interface.png)

`Aware` 是一个具有标识作用的超级接口，实现该接口的 bean 具有被 Spring 容器通知的能力，而被通知的方式就是通过回调，以依赖注入的方式为 bean 设置相应属性，这就是一个典型的依赖注入的使用场景。

参考：[org.springframework.beans.factory.Aware](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/Aware.html)

# 参考

《[Spring in Action, 4th](https://www.manning.com/books/spring-in-action-fourth-edition)》

《[Dependency Injection, Design patterns using Spring and Guice](https://www.manning.com/books/dependency-injection)》

《[IoC模式（依赖、依赖倒置、依赖注入、控制反转）](https://www.cnblogs.com/fuchongjundream/p/3873073.html)》
