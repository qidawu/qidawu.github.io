---
title: Spring Boot 项目搭建
date: 2018-10-01 16:38:00
updated:
tags: Java
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
- 提供统一的依赖版本管理（继承 pom `spring-boot-dependencies`），可以让你在自己的 pom 中引入依赖时省略版本号定义，保障依赖间的兼容性
- 合理的 [resource filtering](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html) 配置
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