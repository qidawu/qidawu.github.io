---
title: Dubbo 协议小结
date: 2018-11-24 10:14:57
updated:
tags: Java
---

# RPC 框架对比

市面上的 RPC 框架功能比较：

![RPC框架功能比较](/img/dubbo/rpc_framework_compare.png)

# 协议对比

|                 | 连接个数 | 连接方式 | 传输协议 | 传输方式     | 序列化                | 适用范围                                                     | 适用场景                                      | 参考                                                         |
| --------------- | -------- | -------- | -------- | ------------ | --------------------- | ------------------------------------------------------------ | --------------------------------------------- | ------------------------------------------------------------ |
| `dubbo://`      | 单连接   | 长连接   | TCP      | NIO 异步传输 | Hessian 二进制序列化  | 传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用 dubbo 协议传输大文件或超大字符串。 | 常规远程服务方法调用                          | [dubbo](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/dubbo.html) |
| `rmi://`        | 多连接   | 短连接   | TCP      | 同步传输     | Java 标准二进制序列化 | 传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。 | 常规远程服务方法调用，与原生RMI服务互操作     |                                                              |
| `hessian://`    | 多连接   | 短连接   | HTTP     | 同步传输     | Hessian二进制序列化   | 传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。 | 页面传输，文件传输，或与原生hessian服务互操作 | [hession](http://hessian.caucho.com/)                        |
| `http://`       | 多连接   | 短连接   | HTTP     | 同步传输     | 表单序列化            | 传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或URL传入参数，暂不支持传文件。 | 需同时给应用程序和浏览器 JS 使用的服务。      |                                                              |
| `webservice://` | 多连接   | 短连接   | HTTP     | 同步传输     | SOAP 文本序列化       |                                                              | 系统集成，跨语言调用                          | [Apache CXF](http://cxf.apache.org/)                         |
| `thrift://`     |          |          |          |              |                       |                                                              |                                               | [Thrift](http://thrift.apache.org/)                          |
| `memcached://`  |          |          |          |              |                       |                                                              |                                               |                                                              |
| `redis://`      |          |          |          |              |                       |                                                              |                                               |                                                              |
| `rest://`       | 多连接   | 可长可短 | HTTP     | 同步传输     | JSON/XML              |                                                              |                                               | [JAX-RS](https://github.com/jax-rs)                          |

# REST 协议小结

根据 dubbox、dubbo REST 官方文档，摘录了使用上的一些注意点：

## 配置总览

### 服务提供端

```xml
<dubbo:protocol name="rest" server="" port="" contextpath="" threads="" iothreads="" keepalive="" accepts="" />
```

| 配置项        | 描述                             | 生效范围             |
| ------------- | -------------------------------- | -------------------- |
| `name`        | 启用 REST 协议                   | all                  |
| `server`      | REST Server 的实现               | all                  |
| `port`        | 端口号                           | all                  |
| `contextpath` | 应用上下文路径                   | all                  |
| `threads`     | 线程池大小                       | jetty、netty、tomcat |
| `iothreads`   | IO worker线程数                  | netty                |
| `keepalive`   | 是否长连接，默认为 `true` 长连接 | netty、tomcat        |
| `accepts`     | 最大的HTTP连接数                 | tomcat               |

目前在dubbo中支持5种嵌入式rest server的实现：

```xml
<!-- rest协议默认选用jetty。jetty是非常成熟的java servlet容器，并和dubbo已经有较好的集成（目前5种嵌入式server中只有jetty、tomcat、tjws，与dubbo监控系统等完成了无缝的集成）。 -->
<dubbo:protocol name="rest" server="jetty"/>
<!-- 在嵌入式tomcat上，REST的性能比jetty上要好得多（参见官网的基准测试），建议在需要高性能的场景下采用tomcat。 -->
<dubbo:protocol name="rest" server="tomcat"/>
<dubbo:protocol name="rest" server="netty"/>
<!-- 轻量级嵌入式server，非常方便在集成测试中快速启动使用，当然也可以在负荷不高的生产环境中使用。注意，tjws is now deprecated -->
<dubbo:protocol name="rest" server="tjws"/>
<dubbo:protocol name="rest" server="sunhttp"/>
```

同时也支持采用外部应用服务器来做rest server的实现：

```xml
<!-- web.xml 参考官网 -->
<dubbo:protocol name="rest" server="servlet"/>
```

### 服务消费端

如果REST服务的消费端也是dubbo系统，可以配置每个消费端的超时时间和HTTP连接数，详情参考官方文档。

## REST 服务提供端

### 标准 Java REST API：JAX-RS

* Dubbox 基于标准的 Java REST API——JAX-RS 2.0（Java API for RESTful Web Services 的简写），提供了接近透明的REST调用支持。由于完全兼容Java标准API，所以为dubbo开发的所有REST服务，未来脱离dubbo或者任何特定的REST底层实现一般也可以正常运行。
* Dubbo的REST调用和dubbo中其它某些RPC不同的是，需要在服务代码中添加JAX-RS的annotation（以及JAXB、Jackson的annotation），如果你觉得这些annotation一定程度“污染”了你的服务代码，你可以考虑编写额外的Facade和DTO类，在Facade和DTO上添加annotation，而Facade将调用转发给真正的服务实现类。当然事实上，直接在服务代码中添加annotation基本没有任何负面作用，而且这本身是Java EE的标准用法，另外JAX-RS和JAXB的annotation是属于java标准，比我们经常使用的spring、dubbo等等annotation更没有vendor lock-in的问题，所以一般没有必要因此而引入额外对象。
* JAX-RS与Spring MVC的对比：
  * JAX-RS 相对更适合纯粹的服务化应用，也就是传统Java EE中所说的中间层服务。
  * 在dubbo应用中，我想很多人都比较喜欢直接将一个本地的spring service bean（或者叫manager之类的）完全透明的发布成远程服务，则这里用JAX-RS是更自然更直接的，不必额外的引入MVC概念。
* 就学习 JAX-RS 来说，一般主要掌握其各种 annotation 的用法即可。参考：
  * http://docs.oracle.com/javaee/7/tutorial/doc/jaxrs.htm
  * http://www.ibm.com/developerworks/cn/java/j-lo-jaxrs/

### HTTP POST/GET 的实现

* REST服务中虽然建议使用HTTP协议中四种标准方法POST、DELETE、PUT、GET来分别实现常见的“增删改查”，但实际中，我们一般情况直接用POST来实现“增改”，GET来实现“删查”即可（DELETE和PUT甚至会被一些防火墙阻挡）。

### JSON、XML 等多数据格式的支持

* 在一个REST服务同时对多种数据格式支持的情况下，根据JAX-RS标准，一般是通过HTTP中的MIME header（content-type和accept）来指定当前想用的是哪种格式的数据。
* 目前业界普遍使用的方式，是使用一个URL后缀（.json和.xml）来指定想用的数据格式。比用HTTP Header更简单直观。Twitter、微博等的REST API都是采用这种方式。

### 定制序列化

* Dubbo中的REST实现是用JAXB做XML序列化，用Jackson做JSON序列化，所以在对象上添加JAXB或Jackson的annotation即可以定制映射。更多资料请参考JAXB和Jackson的官方文档。
* 由于JAX-RS的实现一般都用标准的JAXB（Java API for XML Binding）来序列化和反序列化XML格式数据，所以我们需要为每一个要用XML传输的对象添加一个类级别的JAXB annotation `@XmlRootElement`，否则序列化将报错。

### 添加自定义的 Filter、Interceptor 等

* JAX-RS标准的 `Filter` 和 `Interceptor` 可以对请求与响应过程做定制化的拦截处理。

### 添加自定义的 Exception 处理

* JAX-RS标准的 `ExceptionMapper`，可以用来定制特定exception发生后应该返回的HTTP响应。

## REST 服务消费端

### 场景1：非 dubbo 的消费端调用 dubbo 的 REST 服务（non-dubbo > dubbo）

* 使用标准的JAX-RS Client API或者特定REST实现的Client API来调用REST服务。当然，在java中也可以直接用自己熟悉的比如HttpClient，FastJson，XStream等等各种不同技术来实现REST客户端。

### 场景2：dubbo 消费端调用 dubbo 的 REST 服务（dubbo > dubbo）

- dubbo消费端调用dubbo的REST服务，这种场景下必须把JAX-RS的annotation添加到服务接口上，这样在dubbo在消费端才能共享相应的REST配置信息，并据之做远程调用。
- dubbo的REST支持采用Java标准的[bean validation annotation（JSR 303)](http://beanvalidation.org/)来做输入校验。为了和其他dubbo远程调用协议保持一致，在rest中作校验的annotation必须放在服务的接口上，这样至少有一个好处是，dubbo的消费端可以共享这个接口的信息，dubbo消费端甚至不需要做远程调用，在本地就可以完成输入校验。

# 参考

http://dubbo.apache.org/zh-cn/docs/user/references/protocol/rest.html

https://github.com/dangdangdotcom/dubbox

https://dangdangdotcom.github.io/dubbox/rest.html

https://mvnrepository.com/artifact/com.gaosi/dubbox

Dubbox fork from Dubbo，目前只发布了一个版本：2.8.4