---
title: Spring bean 生命周期与作用域总结
date: 2017-06-03 22:20:33
updated:
tags: Spring
---

# Bean 的生命周期

Spring Bean Factory 负责管理 bean 的生命周期，可以分为三个阶段：

![Spring Bean Life Cycle](/img/spring/spring-bean-life-cycle.png)

初始化阶段：

* Instantiation：Spring 启动，查找并加载需要被 Spring 管理的bean，进行 Bean 的实例化，调用构造方法。
* Populate Properties：属性注入，包括引用的 Bean 和值，调用 setter 方法。
* 调用该 Bean 实现的各种生命周期回调接口。

就绪阶段：

* 此时，Bean 已经准备就绪，可以被应用程序使用了。它们将一直驻留在应用上下文中，直到应用上下文被销毁。

销毁阶段：

* 调用该 Bean 实现的各种生命周期回调接口。

## 初始化和销毁方法

Spring 框架提供了以下三种方式指定 bean 生命周期的初始化和销毁回调方法：

* 实现 Spring 框架的回调接口：
  * `org.springframework.beans.factory.InitializingBean`
  * `org.springframework.beans.factory.DisposableBean`
* 在 Spring bean 配置文件或 `@Bean` 注解中指定以下属性：
  * `@Bean(initMethod="xxx" destroyMethod="xxx")`
  * `<bean init-method="xxx" destroy-method="xxx" />`
* 使用 JavaEE 规范 `javax.annotation` 包中提供的注解：
  - `@PostConstruct`
  - `@PreDestroy`

## Aware 接口

在日常的开发中，我们经常需要用到 Spring 容器本身的功能资源，可以通过 Spring 提供的一系列 `Aware (org.springframework.beans.factory)` 子接口来实现具体的功能。`Aware` 是一个具有标识作用的超级接口，实现该接口的 bean 具有被 Spring 容器通知的能力，而被通知的方式就是通过回调，以依赖注入的方式为 bean 设置相应属性，这是一个典型的依赖注入的使用场景。`Aware` 接口的继承关系如下：

![Aware 接口](/img/spring/aware_interface.png)

这些 `*Aware` 子接口在 Spring Bean  的生命周期中被回调的顺序如下：

1. `BeanNameAware (org.springframework.beans.factory)`
2. `BeanClassLoaderAware (org.springframework.beans.factory)`
3. `BeanFactoryAware (org.springframework.beans.factory)`
4. `EnvironmentAware (org.springframework.context)`
5. `EmbeddedValueResolverAware (org.springframework.context)`
6. `ResourceLoaderAware (org.springframework.context)`
7. `ApplicationEventPublisherAware (org.springframework.context)`
8. `MessageSourceAware (org.springframework.context)`
9. `ApplicationContextAware (org.springframework.context)`

![](/img/spring/spring-bean-lifecycle-2.jpg)

# Bean 的作用域

使用 `@Scope` 注解定义 bean 的作用域，它可以与 `@Component` 或 `@Bean` 一起使用：

* 单例(Singleton):在整个应用中，只创建bean的一个实例。
* 原型(Prototype):每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。
* 会话(Session):在Web应用中，为每个会话创建一个bean实例。
* 请求(Rquest):在Web应用中，为每个请求创建一个bean实例。

# 参考

[org.springframework.beans.factory.Aware](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/Aware.html)