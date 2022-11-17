---
title: Spring Boot 外部化配置总结
date: 2017-08-05 16:38:00
updated:
tags: [Java, Maven, Spring]
typora-root-url: ..
---

# Externalized Configuration

更多信息参考：[7.2 Externalized Configuration](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config)

> Spring Boot lets you externalize your configuration so that you can **work with the same application code in different environments**. You can use a variety of external configuration sources, include Java properties files, YAML files, environment variables, and command-line arguments.
>
> Property values can be injected directly into your beans by 
>
> * using the `@Value` annotation, accessed through Spring’s `Environment` abstraction, 
> * or be [bound to structured objects](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.typesafe-configuration-properties) through `@ConfigurationProperties`.
>
> Spring Boot uses a very particular `PropertySource` order that is designed to allow sensible overriding of values. **Later property sources can override the values defined in earlier ones.** Sources are considered in the following order:
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

[7.2.3. External Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config.files)

> Spring Boot will automatically find and load `application.properties` and `application.yaml` files from the following locations when your application starts:
>
> 1. From the classpath
>    1. The classpath root
>    2. The classpath `/config` package
> 2. From the current directory
>    1. The current directory
>    2. The `config/` subdirectory in the current directory
>    3. Immediate child directories of the `config/` subdirectory
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

## 属性注入

# Common Application Properties

[Appendix A: Common Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix.application-properties)

# 参考

《Spring Boot in Action》

[Spring Boot Tomcat 配置](https://yq.aliyun.com/articles/619390)
