---
title: 《Spring Boot 实战》笔记
date: 2018-10-07 22:30:39
updated:
tags: Java
---

Spring Boot 为基于 Spring 的项目开发带来了很多新功能，例如简化了部署方式、提供了检视应用状态的能力（Actuator），但它最主要的两个目标如下：

1. 简化配置

> 虽然 Spring 的组件代码是轻量级的，但它的配置却是**重量级**的。一开始，Spring 用 XML 配置，而且是很多 XML 配置。Spring 2.5 引入了基于注解的组件扫描，消除了大量针对应用程序自身组件的显式 XML 配置。Spring 3.0 引入了基于 Java 的配置，这是一种类型安全的可重构配置方式，可以代替 XML。
>
> 所有这些配置都代表了开发时的损耗。因为在思考 Spring 特性配置和解决业务问题之间需要进行**思维切换**，所以写配置挤占了写应用程序逻辑的时间。

1. 依赖管理

> 依赖管理也是一种损耗，添加依赖不是写应用程序代码。一旦选错了依赖的版本，随之而来的不兼容问题毫无疑问会是生产力杀手。决定项目里要用哪些库就已经够让人头痛的了，你还要知道这些库的哪个版本和其它库不会有冲突。

# 自动配置

搭建项目后，可以看下其中的 `ReadingListApplication`，它在 Spring Boot 应用程序里有两个作用：配置和启动引导。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ReadingListApplication {
    public static void main(String[] args) {
        SpringApplication.run(ReadingListApplication.class, args);
    }
}
```

`@SpringBootApplication` 由几个注解组成：

- `@Configuration` 标明该类使用 Spring 基于 Java 的配置。

  - 如果你的应用程序需要 Spring Boot 自动配置以外的其它 Spring 配置，一般来说，最好把它写到一个单独的 `@Configuration` 标注的类里。（组件扫描会发现并使用这些类的。）
  - 极度简单的情况下，可以把自定义配置加入 `@SpringBootApplication` 所处的类中。

- `@ComponentScan` 启用组件扫描，自动发现并注册为 Spring 应用程序上下文里的 Bean。

  - > Either `basePackageClasses` or `basePackages` (or its alias value) may be specified to define specific packages to scan. If specific packages are not defined, scanning will occur from the package of the class that declares this annotation. 

- `@EnableAutoConfiguration` 开启 Spring Boot 的自动配置，让你不用再写成篇的配置。

- `@EnableWebMvc`



# 起步依赖

- 起步依赖其实就是特殊的 Maven 依赖和 Gradle 依赖，利用了**传递依赖**解析，把常用库聚合在一起，组成了几个为特定功能而定制的依赖。
- 比起减少依赖数量，起步依赖还引入了一些微妙的变化。向项目中添加了某个起步依赖，实际上指定了应用程序所需的**一类功能**。
- 起步依赖引入的库的版本兼容性都是**经过测试**的，可以放心使用。

# 单元测试



# Actuator



# 部署方式

## 打包

# 开发者工具

