---
title: Spring Boot 入门总结
date: 2017-08-01 16:38:00
updated:
tags: [Java, Maven, Spring]
typora-root-url: ..
---

# Roadmap

截止 2021.9，Spring Boot 的最新版本情况：

|                             版本                             |  发布时间  | 停止维护时间 |
| :----------------------------------------------------------: | :--------: | :----------: |
|                            2.7.0                             |  2022/05   |      -       |
|                            2.6.0                             | 2021/12/18 |      -       |
| [2.5.x](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247530679&idx=1&sn=32b3a0458c269df212556c1620ee93fa&scene=21#wechat_redirect) | 2021/05/20 |  2023/02/20  |
| [2.4.x](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247514253&idx=1&sn=7bb0d45039d35740981e98858f550647&scene=21#wechat_redirect) | 2020/12/12 |  2022/08/12  |
| [2.3.x](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247496866&idx=2&sn=6304a068d392d05ab86caf0c4fe2c514&scene=21#wechat_redirect) | 2020/05/15 |  2022/02/15  |
| [2.2.x](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247489065&idx=1&sn=3efba2d3ab5894f30158507154db6af5&scene=21#wechat_redirect) |  2019/10   |  已停止维护  |
| [2.1.x](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247487615&idx=1&sn=87e870f0d3e9b359bb3912d39d8a4c45&scene=21#wechat_redirect) |  2018/10   |  已停止维护  |
| [2.0.x](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247485347&idx=1&sn=ff480a238c6cb84c1bb8363272781b43&scene=21#wechat_redirect) |  2018/03   |  已停止维护  |
| [1.5.x](https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247485192&idx=1&sn=915e013ebb0ce4166c685d1747fdd47c&scene=21#wechat_redirect) |  2017/01   |  已停止维护  |

# Getting Started

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

# Spring Boot 组成

Spring Boot 的[各个子项目](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project)组成及结构如下：

```
spring-boot-dependencies (BOM)
  spring-boot-parent (Parent POM)
    spring-boot
    spring-boot-autoconfigure
    spring-boot-starters (Parent module)
      spring-boot-starter (Core starter, including auto-configuration support, logging and YAML)
      spring-boot-starter-parent (Parent pom providing dependency and plugin management for applications
		built with Maven)
      spring-boot-starter-web
      ...
    spring-boot-test
    spring-boot-test-autoconfigure
    spring-boot-actuator
    spring-boot-actuator-autoconfigure
    spring-boot-devtools
    spring-boot-cli
    spring-boot-docs
    spring-boot-tools (Parent module)
      spring-boot-autoconfigure-processor
      spring-boot-configuration-processor
      spring-boot-maven-plugin
      spring-boot-gradle-plugin
      ...
```

## 项目管理

- [`spring-boot-dependencies`](https://github.com/spring-projects/spring-boot/tree/main/spring-boot-project/spring-boot-dependencies)
  
  - BOM (Bill of Materials)，用于定义和统一管理 Sprint Boot 的各个依赖版本号。
  
  - 业务项目可以通过 `dependencyManagement` 引入该依赖，解决**单继承问题**。
  
  - 业务项目可以覆盖版本号如下（但不建议）：
  
    ```XML
    <properties>
      <log4j2.version>2.16.0</log4j2.version>
    </properties>
    ```
  
- [`spring-boot-parent`](https://github.com/spring-projects/spring-boot/tree/main/spring-boot-project/spring-boot-parent)
  
  - Spring Boot 各个依赖的父 POM，用于构建配置。
  - 继承自 `spring-boot-dependencies`。

## 核心依赖

- `spring-boot` Spring Boot 的核心工程。
- [`spring-boot-autoconfigure`](https://github.com/spring-projects/spring-boot/tree/main/spring-boot-project/spring-boot-autoconfigure) 实现 Spring Boot 自动配置的关键，常用的包含：
  - 自动配置总开关 `@EnableAutoConfiguration`
  - 各种自动配置类 `*AutoConfiguration`
  - 各种外部化配置属性类 `*Properties`
  - 各种条件化注解类 `@ConditionOn*`
- [`spring-boot-starters`](https://github.com/spring-projects/spring-boot/tree/main/spring-boot-project/spring-boot-starters) 起步依赖的父 POM
  - Spring Boot 提供的众多起步依赖，用于降低项目依赖的复杂度，清单详见：[Starters](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-starter)，例如：
    - [`spring-boot-starter`](https://github.com/spring-projects/spring-boot/tree/main/spring-boot-project/spring-boot-starters/spring-boot-starter) 核心起步依赖，包括自动配置支持、日志、YAML 依赖
    - [`spring-boot-starter-parent`](https://github.com/spring-projects/spring-boot/tree/main/spring-boot-project/spring-boot-starters/spring-boot-starter-parent) 业务项目的**父 POM**，继承自 `spring-boot-dependencies`
    - `spring-boot-starter-web` WEB 开发相关起步依赖
    - `spring-boot-starter-test` 测试相关起步依赖
    - ...
  - 起步依赖本质上就是特殊的 Maven 依赖和 Gradle 依赖，利用了**传递依赖**解析，把常用库聚合在一起，组成了几个为特定功能而定制的依赖。
  - 比起减少依赖数量，起步依赖还引入了一些微妙的变化。向项目中添加了某个起步依赖，实际上指定了应用程序所需的**一类功能**。
  - 起步依赖引入的库的版本兼容性都是**经过测试**的，可以放心使用。

## 工具或插件

- `spring-boot-test`、`spring-boot-test-autoconfigure`
  - 提供一系列测试支持，常用的如：`@SpringBootTest`、mock、web 支持。
- `spring-boot-actuator`、`spring-boot-actuator-autoconfigure`
  - 包含许多额外的特性，以帮助你通过 HTTP 或 JMX 端点来监控和管理生产环境的应用程序。包括以下特性（详见[用户手册](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready)）：
    - **Endpoints** Actuator endpoints allow you to monitor and interact with your application. Spring Boot includes a number of built-in endpoints and you can also add your own. For example the `health` endpoint provides basic application health information. Run up a basic application and look at `/actuator/health`.
    - **Metrics** Spring Boot Actuator provides dimensional metrics by integrating with [Micrometer](https://micrometer.io/).
    - **Audit** Spring Boot Actuator has a flexible audit framework that will publish events to an `AuditEventRepository`. Once Spring Security is in play it automatically publishes authentication events by default. This can be very useful for reporting, and also to implement a lock-out policy based on authentication failures.
- `spring-boot-devtools`
  - 热部署、静态资源 livereload 等等。
- `spring-boot-tools` 工具集的父 POM。为 Spring Boot 开发者提供的常用工具集。例如：
  - `spring-boot-maven-plugin` 插件
  - `spring-boot-gradle-plugin` 插件
- `spring-boot-cli`
  - 命令行工具。

# POM 配置

参考：[Build Tool Plugins](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins.html)

## 继承 spring-boot-starter-parent

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
- 提供统一的依赖版本管理（继承自 `spring-boot-dependencies` BOM），可以让你在自己的 pom 中引入依赖时省略版本号定义，保障依赖间的兼容性
- An execution of the [`repackage` goal](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/repackage-mojo.html) with a `repackage` execution id.
- 合理的 plugin configuration 配置
- 合理的 [resource filtering](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html) 配置（`application.properties` and `application.yml` including profile-specific files）

## 组合 spring-boot-dependencies

不是每个人都喜欢继承 `spring-boot-starter-parent` POM 项目。每个公司可能都拥有自己的标准父项目，或者你更愿意明确声明所有 Maven 配置。

即使如此，你仍然可以以组合方式引入 `scope=import`  的 `spring-boot-dependencies` BOM 项目，来享受依赖管理的好处。如下所示：

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

这种组合方式能解决 Maven 单继承问题。

## 引入 spring-boot-maven-plugin

> The Spring Boot Maven Plugin provides Spring Boot support in [Apache Maven](https://maven.org/). It allows you to package executable jar or war archives, run Spring Boot applications, generate build information and start your Spring Boot application prior to running integration tests.

`spring-boot-maven-plugin` 插件内置[几个 goal](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals)，如下：

![spring-boot-maven-plugin](/img/spring/spring-boot/spring-boot-maven-plugin.png)

| Goal                                                         | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [spring-boot:build-image](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals-build-image) | Package an application into a OCI image using a buildpack.   |
| [spring-boot:build-info](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals-build-info) | Generate a `build-info.properties` file based on the content of the current `MavenProject`. |
| [spring-boot:help](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals-help) | Display help information on spring-boot-maven-plugin. Call `mvn spring-boot:help -Ddetail=true -Dgoal=<goal-name>` to display parameter details. |
| [spring-boot:repackage](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals-repackage) | Repackage existing JAR and WAR archives so that they can be executed from the command line using `java -jar`. With `layout=NONE` can also be used simply to package a JAR with nested dependencies (and no main class, so not executable). |
| [spring-boot:run](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals-run) | Run an application in place.                                 |
| [spring-boot:start](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals-start) | Start a spring application. Contrary to the `run` goal, this does not block and allows other goals to operate on the application. This goal is typically used in integration test scenario where the application is started before a test suite and stopped after. |
| [spring-boot:stop](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals-stop) | Stop an application that has been started by the "start" goal. Typically invoked once a test suite has completed. |

### 项目运行方式一

Goal `spring-boot:run` 用于快速编译并运行 Spring Boot 应用，常用于本地开发环境。命令：

```bash
$ mvn spring-boot:run
```

### 项目运行方式二

Goal `spring-boot:repackage` 用于重新打包现有的 jar/war 包，以便可以通过 `java -jar` 命令运行，常用于部署环境。命令：

```bash
# `clean` Phase 会清理 target 目录
# `package` Phase 先将项目打包成一个 jar/war 包
# `spring-boot:repackage` Goal 再重新打成 jar 包
$ mvn clean package spring-boot:repackage
```

⚠️ 如果未 `mvn package` 就直接 `mvn spring-boot:repackage` 会打包失败：

```
org.springframework.boot:spring-boot-maven-plugin:X.X.X.RELEASE:repackage failed: Source file must not be null 
```

[Executing `spring-boot:repackage` Goal During Maven's `package` Phase](https://www.baeldung.com/spring-boot-repackage-vs-mvn-package#4-executing-spring-bootrepackage-goal-during-mavens-package-lifecycle)

> We can configure the Spring Boot Maven Plugin in our *pom.xml* to `repackage` the artifact during the `package` phase of the Maven lifecycle. In other words, when we execute `mvn package`, the `spring-boot:repackage` will be automatically executed.
>
> The configuration is pretty straightforward. We just add the `repackage` goal to an *execution* element:
>
> ```xml
> <build>
>     <finalName>${project.artifactId}</finalName>
>     <plugins>
>         <plugin>
>             <groupId>org.springframework.boot</groupId>
>             <artifactId>spring-boot-maven-plugin</artifactId>
>             <executions>
>                 <execution>
>                     <goals>
>                         <goal>repackage</goal>
>                     </goals>
>                 </execution>
>             </executions>
>         </plugin>
>     </plugins>
> </build>
> ```

参考：

* https://www.baeldung.com/spring-boot-repackage-vs-mvn-package
* https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html

重新打包后的 jar 包内容如下：

![repackaged jar file](/img/spring/spring-boot/spring-boot-repackaged-jar-file.png)

重新打包后的 jar 包会内嵌一个 Servlet 容器，你可以像运行任何 Java 应用程序一样运行它：

```bash
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

也可以指定 Java System properties 运行，例如：

```bash
$ java -jar -Dspring.profiles.active=prod target/myapplication-0.0.1-SNAPSHOT.jar
```

### 项目运行方式三

通过在 jar 中添加启动脚本，为 *nix 系统制作一个完全可执行的 jar。常用于生产环境。

参考：[14.2 Installing Spring Boot Applications](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#deployment.installing)

> In addition to running Spring Boot applications by using `java -jar`, it is also possible to make **fully executable** applications for Unix systems. A fully executable jar can be executed like any other executable binary or it can be [registered with `init.d` or `systemd`](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#deployment.installing.nix-services). This helps when installing and managing Spring Boot applications in common production environments.
>
> > ⚠️ Caution
> >
> > Fully executable jars work by embedding an extra script at the front of the file. It is recommended that you make your jar or war fully executable only if you intend to execute it directly, rather than running it with `java -jar` or deploying it to a servlet container.
>
> To create a ‘fully executable’ jar with Maven, use the following plugin configuration:
>
> ```XML
> <plugin>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-maven-plugin</artifactId>
>     <configuration>
>         <executable>true</executable>
>     </configuration>
> </plugin>
> ```
>
> You can then run your application by typing `./my-application.jar` (where `my-application` is the name of your artifact). The directory containing the jar is used as your application’s working directory.

参考：[Optional parameter `executable` of spring-boot:repackage](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals-repackage-parameters-details-executable)

> Make a fully executable jar for *nix machines by prepending a **launch script** to the jar.

#### Unix/Linux Services

更多信息参考：[14.2.2. Unix/Linux Services](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#deployment.installing.nix-services)

* Installation as an `init.d` Service (System V)

* Installation as a `systemd` Service

  > `systemd` is the successor of the System V init system and is now being used by many modern Linux distributions.

* Customizing the Startup Script

  * [Customizing the Start Script When It Is Written](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#deployment.installing.nix-services.script-customization.when-written)

  * [Customizing a Script When It Runs](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#deployment.installing.nix-services.script-customization.when-running)

    > For items of the script that need to be customized *after* the jar has been written, you can use environment variables or a [config file](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment.html#deployment.installing.nix-services.script-customization.when-running.conf-file).
    >
    > The [following environment properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#deployment.installing.nix-services.script-customization.when-running) are supported with the default script.

    > With the exception of `JARFILE` and `APP_NAME`, the settings listed in the preceding section can be configured by using a `.conf` file. **The file is expected to be next to the jar file and have the same name but suffixed with `.conf` rather than `.jar`.** For example, a jar named `/var/myapp/myapp.jar` uses the configuration file named `/var/myapp/myapp.conf`, as shown in the following example:
    >
    > ```
    > MODE=service
    > JAVA_OPTS="-Xmx1024M -Dspring.profiles.active=prod"
    > PID_FOLDER=./
    > LOG_FOLDER=./
    > ```

##### 脚本命令

当设置 `MODE=service`，`./my-application.jar` 可执行命令如下：

> You can explicitly set it to `service` so that the `stop|start|status|restart` commands work or to `run` if you want to run the script in the foreground.

* `status` 查看运行状态和 PID（Started、Running、Stoped、Not running）
* `stop` 优雅停止应用
* `force-stop` 强制停止应用
* `start` 启动应用
* `restart` 重启应用

# 外部化配置

更多信息参考：[Externalized Configuration](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config)

> Spring Boot lets you externalize your configuration so that you can **work with the same application code in different environments**. You can use a variety of external configuration sources, include Java properties files, YAML files, environment variables, and command-line arguments.
>
> Property values can be injected directly into your beans by 
>
> * using the `@Value` annotation, accessed through Spring’s `Environment` abstraction, 
> * or be [bound to structured objects](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.typesafe-configuration-properties) through `@ConfigurationProperties`.
>
> Spring Boot uses a very particular `PropertySource` order that is designed to allow sensible overriding of values. Properties are considered in the following order (with values from **lower items overriding earlier ones**):
>
> 1. Default properties (specified by setting `SpringApplication.setDefaultProperties`).
> 2. [`@PropertySource`](https://docs.spring.io/spring-framework/docs/5.3.20/javadoc-api/org/springframework/context/annotation/PropertySource.html) annotations on your `@Configuration` classes.
> 3. ⭐️ Config data (such as `application.properties` files).
> 4. A `RandomValuePropertySource` that has properties only in `random.*`.
> 5. ⭐️ OS environment variables (`System.getenv()`).
> 6. ⭐️ Java System properties (`System.getProperties()`) (that is, arguments starting with `-D`, such as ` -Dspring.profiles.active=prod`).
> 7. ⭐️ JNDI attributes from `java:comp/env`.
> 8. `ServletContext` init parameters.
> 9. `ServletConfig` init parameters.
> 10. Properties from `SPRING_APPLICATION_JSON` (inline JSON embedded in an environment variable or system property).
> 11. ⭐️ Command line arguments. (that is, arguments starting with `--`, such as `--server.port=9000`)
> 12. `properties` attribute on your tests. Available on [`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.7.0/api/org/springframework/boot/test/context/SpringBootTest.html) and the [test annotations for testing a particular slice of your application](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.testing.spring-boot-applications.autoconfigured-tests).
> 13. [`@TestPropertySource`](https://docs.spring.io/spring-framework/docs/5.3.20/javadoc-api/org/springframework/test/context/TestPropertySource.html) annotations on your tests.
> 14. [Devtools global settings properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using.devtools.globalsettings) in the `$HOME/.config/spring-boot` directory when devtools is active.

> ⭐️ Config data files are considered in the following order:
>
> 1. [Application properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.files) packaged inside your jar (`application.properties` and YAML variants).
> 2. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.files.profile-specific) packaged inside your jar (`application-{profile}.properties` and YAML variants).
> 3. [Application properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.files) outside of your packaged jar (`application.properties` and YAML variants).
> 4. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.files.profile-specific) outside of your packaged jar (`application-{profile}.properties` and YAML variants).

> It is recommended to stick with one format for your entire application. If you have configuration files with both `.properties` and `.yml` format in the same location, `.properties` takes precedence.

## External Application Properties

> Spring Boot will automatically find and load `application.properties` and `application.yaml` files from the following locations when your application starts:
>
> 1. The classpath root
> 2. The classpath `/config` package
>
> The list is ordered by precedence (with values from **lower items overriding earlier ones**). Documents from the loaded files are added as `PropertySources` to the Spring `Environment`.

`spring.config.name` 修改配置文件名称：

> If you do not like `application` as the configuration file name, you can switch to another file name by specifying a `spring.config.name` environment property.
>
> For example, to look for `myproject.properties` and `myproject.yaml` files you can run your application as follows:
>
> ```shell
> $ java -jar myproject.jar --spring.config.name=myproject
> ```

`spring.config.location` 引用外部配置文件：

> You can also refer to an explicit location by using the `spring.config.location` environment property. This property accepts a comma-separated list of one or more locations to check.
>
> The following example shows how to specify two distinct files:
>
> ```shell
> $ java -jar myproject.jar --spring.config.location=\
>  optional:classpath:/default.properties,\
>  optional:classpath:/override.properties
> ```
>
> Use the prefix `optional:` if the [locations are optional](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.files.optional-prefix) and you do not mind if they do not exist.

## Common Application Properties

https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties

## 例子

### 配置嵌入式服务器

Spring Boot 集成了 Tomcat、Jetty 和 Undertow，极大便利了项目部署。下面介绍一些常用配置：

```properties
server.port=8080 # Server HTTP port.
server.context-path= # Context path
```

#### Tomcat

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

#### Undertow

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

https://github.com/spring-projects/spring-boot

https://docs.spring.io/spring-boot/docs/current/

https://docs.spring.io/spring-boot/docs/current/reference/html/index.html

* [6.1.5. Using Spring Boot Starters](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-starter)
* [7.2. Externalized Configuration](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config)
* [14.2 Installing Spring Boot Applications](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#deployment.installing)
  * [14.2.2 Unix/Linux Services - Customizing the Startup Script](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#deployment.installing.nix-services.script-customization)

[Spring Boot 的 16 条最佳实践](https://mp.weixin.qq.com/s/alLpto0ZCnkv7ew2tWLtKQ)

[Spring Boot Tomcat 配置](https://yq.aliyun.com/articles/619390)

