---
title: Spring Boot 入门总结
date: 2018-10-01 16:38:00
updated:
tags: [Java, Spring]
---

# 搭建项目

有几种方式可以搭建基于 Spring Boot 的项目：

1. [Spring Initializr](https://start.spring.io/) 在线生成项目

  > Spring Initializr 从本质上来说就是一个Web应用程序，它能为你生成 Spring Boot 项目结构。虽然不能生成应用程序代码，但它能为你提供一个基本的项目结构，以及一个用于构建代码的 Maven 或 Gradle 构建说明文件。你只需要写应用程序的代码就好了。

2. [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/html/cli.html) 命令行工具，下载地址[点我](http://repo.spring.io/release/org/springframework/boot/spring-boot-cli/)，命令如下：

  ```bash
  $ spring init -dweb,data-jpa,h2,thymeleaf --build gradle readingList
  ```

3. [Spring Tool Suite](https://spring.io/tools/sts)，一个官方基于 Eclipse 定制的 IDE，用法参考：《[STS 创建第一个 Spring Boot 项目](http://blog.csdn.net/linabc123000/article/details/68954236)》

4. IntelliJ IDEA

   1. 社区版：离线安装 [Spring Assistant](http://plugins.jetbrains.com/plugin/10229-spring-assistant) 插件（在线安装方式被墙）
   2. 收费版：直接使用 Spring Initializr 插件

# 项目组成

Spring Boot 的[各个子项目](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project)组成如下：

- `spring-boot-dependencies`
  - 用于定义和统一管理 Sprint Boot 的各个依赖版本号，继承自 `spring-boot-build`。
- `spring-boot-parent`
  - Spring Boot 父 pom.xml（只包含这一个文件），用于构建配置。继承自 `spring-boot-dependencies`，且被其它子项目所继承。
- `spring-boot`
  - Spring Boot 的核心工程。
- `spring-boot-starters`
  - Spring Boot 提供的众多起步依赖，用于降低项目依赖的复杂度，清单详见：[Starters](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-starter)
  - 起步依赖本质上就是特殊的 Maven 依赖和 Gradle 依赖，利用了**传递依赖**解析，把常用库聚合在一起，组成了几个为特定功能而定制的依赖。
  - 比起减少依赖数量，起步依赖还引入了一些微妙的变化。向项目中添加了某个起步依赖，实际上指定了应用程序所需的**一类功能**。
  - 起步依赖引入的库的版本兼容性都是**经过测试**的，可以放心使用。
- `spring-boot-autoconfigure`
  - 实现 Spring Boot 自动配置的关键，常用的包含：
    - 自动配置总开关`@EnableAutoConfiguration`
    - 自动配置类 `*AutoConfiguration`
    - 外部化配置属性类 `*Properties`
    - 条件化注解类 `@ConditionOn*`
- `spring-boot-test`、`spring-boot-test-autoconfigure`
  - 提供一系列测试支持，常用的如：`@SpringBootTest`、mock、web 支持。
- `spring-boot-actuator`、`spring-boot-actuator-autoconfigure`
  - 包含许多额外的特性，以帮助你通过 HTTP 或 JMX 端点来监控和管理生产环境的应用程序。包括以下特性（详见[用户手册](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready)）：
    - **Endpoints** Actuator endpoints allow you to monitor and interact with your application. Spring Boot includes a number of built-in endpoints and you can also add your own. For example the `health` endpoint provides basic application health information. Run up a basic application and look at `/actuator/health`.
    - **Metrics** Spring Boot Actuator provides dimensional metrics by integrating with [Micrometer](https://micrometer.io/).
    - **Audit** Spring Boot Actuator has a flexible audit framework that will publish events to an `AuditEventRepository`. Once Spring Security is in play it automatically publishes authentication events by default. This can be very useful for reporting, and also to implement a lock-out policy based on authentication failures.
- `spring-boot-devtools`
  - 热部署、静态资源 livereload 等等。
- `spring-boot-tools` 为 Spring Boot 开发者提供的常用工具集。例如：
  - `spring-boot-maven-plugin`
  - `spring-boot-gradle-plugin`
- `spring-boot-cli`
  - 命令行工具。

# POM 配置

* 方式一：继承 `spring-boot-starter-parent`
* 方式二：如果已经有父项目，组合 `spring-boot-dependencies`

参考：[Build Systems - Maven](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-build-systems.html#using-boot-maven)

## 继承方式

Maven 用户可以继承 `spring-boot-starter-parent` POM 项目以获得合理的默认配置：

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
</parent>
```

你只需要指定 parent 的 `<version>` 版本号。之后如果你需要引入其它起步依赖，你可以安全的省略起步依赖的 `<version>` 版本号，parent 会统一管理。

父项目还提供了以下功能：

- Java 1.8 作为默认的编译器级别
- UTF-8 源码编码
- 提供统一的依赖版本管理（继承自 Maven POM `spring-boot-dependencies`），可以让你在自己的 pom 中引入依赖时省略版本号定义，保障依赖间的兼容性
- An execution of the [`repackage` goal](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/repackage-mojo.html) with a `repackage` execution id.
- 合理的 [resource filtering](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html) 配置（`application.properties` and `application.yml` including profile-specific files）
- 合理的 plugin configuration 配置

## 组合方式

不是每个人都喜欢继承 `spring-boot-starter-parent` POM 项目。每个公司可能拥有自己的标准父项目，或者你更愿意明确声明所有 Maven 配置。

即使如此，你仍然可以通过以组合方式使用 `scope=import`  的 `spring-boot-dependencies` 依赖来享受依赖管理的好处，如下所示：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.0.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

# Maven 插件

`spring-boot-maven-plugin` 插件含有几个 goal，主要提供两个功能：

* `spring-boot:run` 用于快速编译并运行。
* `spring-boot:repackage` 用于将项目**打包**成一个可执行的 jar/war 包，使用命令：`mvn package spring-boot:repackage`。

参考：

* [Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-maven-plugin.html)
* https://docs.spring.io/spring-boot/docs/current/maven-plugin/

# 运行项目

## 方式一

使用 `spring-boot-maven-plugin` 插件自带的命令 `mvn spring-boot:run`

## 方式二

使用插件打包好的 jar 包内会内嵌一个容器，你可以像运行任何其它应用程序一样运行它：

```bash
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

也可以指定运行参数，例如：

```bash
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar -spring.profiles.active=prod
```

# 参考

《Spring Boot in Action》

https://docs.spring.io/spring-boot/docs/current/

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-starter

https://github.com/spring-projects/spring-boot