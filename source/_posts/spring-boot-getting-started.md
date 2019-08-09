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
$ java -jar -spring.profiles.active=prod target/myapplication-0.0.1-SNAPSHOT.jar
```

# 外部配置

Spring Boot 能从多种属性源获得属性，包括如下几处：

1. 命令行参数
2. `java:comp/env` 里的 JNDI 属性
3. JVM 系统属性
4. 操作系统环境变量
5. 随机生成的带 `random.*` 前缀的属性（在设置其他属性时，可以引用它们，比如 `${random.long}`）
6. 应用程序以外的 `application.properties` 或者 `appliaction.yml` 文件
7. 打包在应用程序内的 `application.properties` 或者 `appliaction.yml` 文件
8. 通过 `@PropertySource` 标注的属性源
9. 默认属性

这个列表按照优先级排序，也就是说，任何在高优先级属性源里设置的属性都会覆盖低优先级的相同属性。例如，命令行参数会覆盖其他属性源里的属性。

`application.properties` 和 `application.yml` 文件能放在以下四个位置：

1. 外置，在相对于应用程序运行目录的 `/config` 子目录里。
2. 外置，在应用程序运行的目录里。
3. 内置，在 `config` 包内。
4. 内置，在 `Classpath` 根目录。

同样，这个列表按照优先级排序。也就是说，`/config` 子目录里的 `application.properties` 会覆盖应用程序 `Classpath` 里的 `application.properties` 中的相同属性。

此外，如果你在同一优先级位置同时有 `application.properties` 和 `application.yml`，那么 `application.yml` 里的属性会覆盖 `application.properties` 里的属性。

如果需要为自己的 Bean 加上外部配置注入，可以使用注解 `@ConfigurationProperties`。

## 配置嵌入式服务器

Spring Boot 集成了 Tomcat、Jetty 和 Undertow，极大便利了项目部署。下面介绍一些常用配置：

```properties
server.port=8080 # Server HTTP port.
server.context-path= # Context path
```

### Tomcat

URI 编码配置：

```properties
server.tomcat.uri-encoding=UTF-8 # Character encoding to use to decode the URI.
```

代理配置：

```properties
server.tomcat.remote-ip-header= # Name of the http header from which the remote ip is extracted. For instance `X-FORWARDED-FOR`
server.tomcat.protocol-header= # Header that holds the incoming protocol, usually named "X-Forwarded-Proto".
server.tomcat.port-header=X-Forwarded-Port # Name of the HTTP header used to override the original port value.
```

Socket 连接限制及等待超时时间：

```properties
server.tomcat.max-connections= # Maximum number of connections that the server will accept and process at any given time.
server.connection-timeout= # Time in milliseconds that connectors will wait for another HTTP request before closing the connection. When not set, the connector's container-specific default will be used. Use a value of -1 to indicate no (i.e. infinite) timeout.
```

业务线程池调优：

```properties
server.tomcat.max-threads=0 # Maximum amount of worker threads. Default 200.
server.tomcat.min-spare-threads=0 # Minimum amount of worker threads.
server.tomcat.accept-count= # Maximum queue length for incoming connection requests when all possible request processing threads are in use.
```

### Undertow

```properties
# 设置IO线程数, 它主要执行非阻塞的任务,它们会负责多个连接, 默认设置每个CPU核心一个线程
# 不要设置过大，如果过大，启动项目会报错：打开文件数过多
server.undertow.io-threads=16

# 阻塞任务线程池, 当执行类似servlet请求阻塞IO操作, undertow会从这个线程池中取得线程
# 它的值设置取决于系统线程执行任务的阻塞系数，默认值是IO线程数*8
server.undertow.worker-threads=256

# 以下的配置会影响buffer,这些buffer会用于服务器连接的IO操作,有点类似netty的池化内存管理
# 每块buffer的空间大小,越小的空间被利用越充分，不要设置太大，以免影响其他应用，合适即可
server.undertow.buffer-size=1024

# 每个区分配的buffer数量 , 所以pool的大小是buffer-size * buffers-per-region
server.undertow.buffers-per-region=1024

# 是否分配的直接内存(NIO直接分配的堆外内存)
server.undertow.direct-buffers=true
```

https://www.cnblogs.com/duanxz/p/9337022.html

# 参考

《Spring Boot in Action》

https://docs.spring.io/spring-boot/docs/current/

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-starter

https://github.com/spring-projects/spring-boot

[Spring Boot Tomcat配置](https://yq.aliyun.com/articles/619390)