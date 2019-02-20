---
title: Java SPI 机制及 Dubbo SPI、Spring Factories 对比总结
date: 2019-01-15 22:27:20
updated:
tags: Java
---

# Java SPI

SPI 全称 Service Provider Interface，是 Java 在语言层面为我们提供了一种方便地创建可扩展应用的途径。SPI 提供了一种 JVM 级别的服务发现机制，我们只需要按照 SPI 的要求，在 jar 包中进行适当的配置，JVM 就会在运行时通过懒加载，帮我们找到所需的服务并加载。如果我们一直不使用某个服务，那么它不会被加载，一定程度上避免了资源的浪费。

整体机制图如下：

![Java SPI](/img/java/java-spi.webp)



Java SPI 实际上是“**基于接口的编程＋策略模式＋配置文件**”组合实现的动态加载机制。其使用场景如下：

* JDBC Driver 驱动程序加载，详见：[JDBC Driver 驱动程序总结](/2019/01/23/java-jdbc-driver/)
* 日志门面接口实现类加载，SLF4J 加载不同提供商的日志实现类。
* Spring
  * 对 servlet3.0 规范对 `ServletContainerInitializer` 的实现
  * 自动类型转换 Type Conversion SPI (Converter SPI、Formatter SPI) 等
  * Spring Factories 机制（`SpringFactoriesLoader`）
* Dubbo 通过 SPI 机制加载所有的组件。不过，Dubbo 并未使用 Java 原生的 SPI 机制，而是对其进行了增强，使其能够更好的满足需求。在 Dubbo 中，SPI 是一个非常重要的模块。基于 SPI，我们可以很容易的对 Dubbo 进行拓展。如果大家想要学习 Dubbo 的源码，SPI 机制务必弄懂。

# Dubbo SPI

详见：http://dubbo.apache.org/zh-cn/docs/source_code_guide/dubbo-spi.html

# 总结

|          | Java SPI                         | Spring Factories                                    | Dubbo SPI                                               |
| -------- | -------------------------------- | --------------------------------------------------- | ------------------------------------------------------- |
| 加载类   | `ServiceLoader`                  | `SpringFactoriesLoader`                             | `ExtensionLoader`                                       |
| 加载文件 | `META-INF/services/接口全限定名` | `META-INF/spring.factories`                         | `META-INF/dubbo`                                        |
| 文件内容 | 接口实现类，多值以**换行**分隔   | 通过键值对方式（key=value）配置，多值以**逗号**分隔 | 通过键值对方式（key=value）配置，支持按需加载接口实现类 |
| 接口注解 | /                                | /                                                   | `@SPI`                                                  |

# 参考

https://www.jianshu.com/p/46b42f7f593c