---
title: Dubbo 配置小结
date: 2017-11-15 20:14:05
updated:
tags: Java
typora-root-url: ..
---

本文主要总结 Dubbo 日常使用时的一些常用配置。

# 配置之间的关系

![配置之间的关系](/img/java/dubbo/dubbo-config.jpg)

| XML 配置               | Java Config 配置                             | 配置         | 解释                                                         |
| ---------------------- | -------------------------------------------- | ------------ | ------------------------------------------------------------ |
| `<dubbo:application/>` | `com.alibaba.dubbo.config.ApplicationConfig` | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者           |
| `<dubbo:registry/>`    | `com.alibaba.dubbo.config.RegistryConfig`    | 注册中心配置 | 用于配置连接注册中心相关信息                                 |
| `<dubbo:monitor/>`     | `com.alibaba.dubbo.config.MonitorConfig`     | 监控中心配置 | 用于配置连接监控中心相关信息，可选                           |
| `<dubbo:protocol/>`    | `com.alibaba.dubbo.config.ProtocolConfig`    | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方**被动接受** |
| `<dubbo:provider/>`    | `com.alibaba.dubbo.config.ProviderConfig`    | 提供方配置   | 当 `ProtocolConfig` 和 `ServiceConfig` 某属性没有配置时，采用此缺省值，可选 |
| `<dubbo:service/>`     | `com.alibaba.dubbo.config.ServiceConfig`     | 服务配置     | 用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心。对应注解：`@Service` |
| `<dubbo:consumer/>`    | `com.alibaba.dubbo.config.ConsumerConfig`    | 消费方配置   | 当 `ReferenceConfig` 某属性没有配置时，采用此缺省值，可选    |
| `<dubbo:reference/>`   | `com.alibaba.dubbo.config.ReferenceConfig`   | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心。对应注解：`@Reference` |
| `<dubbo:method/>`      | `com.alibaba.dubbo.config.MethodConfig`      | 方法配置     | 用于 `ServiceConfig` 和 `ReferenceConfig` 指定方法级的配置信息 |
| `<dubbo:argument/>`    | `com.alibaba.dubbo.config.ArgumentConfig`    | 参数配置     | 用于指定方法参数配置                                         |
| `<dubbo:module/>`      | `com.alibaba.dubbo.config.ModuleConfig`      | 模块配置     | 用于配置当前模块信息，可选                                   |

下面是一些 dubbo 配置的总结：

## 协议

![](/img/java/dubbo/dubbo_protocol.png)

## 服务提供者

![](/img/java/dubbo/provider.png)

## 服务消费者

![](/img/java/dubbo/consumer.png)

# 配置覆盖关系

配置覆盖关系：

* 方法级优先，接口级次之，全局配置再次之。
* 如果级别一样，则消费方优先，提供方次之。

规则二是指，所有配置最终都将转换为 URL 表示，并由服务提供方生成，经注册中心传递给消费方。其 URL 格式如下：`protocol://username:password@host:port/path?key=value&key=value`

![dubbo 配置覆盖关系](/img/java/dubbo/dubbo-config-override.jpg)

参考：[XML 配置](http://dubbo.apache.org/zh-cn/docs/user/configuration/xml.html)。

# 属性配置关系

![dubbo 属性覆盖关系](/img/java/dubbo/dubbo-properties-override.jpg)

参考：[属性配置](http://dubbo.apache.org/zh-cn/docs/user/configuration/properties.html)

# 注解配置实践

如果想用现代的 Java Config 替代传统的 XML 配置方式，配置如下：

## 声明组件

* 服务提供方使用 `@Service` 注解暴露服务

  ```java
  import com.alibaba.dubbo.config.annotation.Service;
   
  @Service(timeout = 5000)
  public class AnnotateServiceImpl implements AnnotateService { 
      // ...
  }
  ```

* 服务消费方使用 `@Reference` 注解引用服务

  ```java
  import com.alibaba.dubbo.config.annotation.Reference;
  
  public class AnnotationConsumeService {
      @Reference
      public AnnotateService annotateService;
      
      // ...
  }
  ```

## 开启组件扫描

* 2.5.7 (Nov, 2017) 以上版本，使用 `@DubboComponentScan` 指定 dubbo 组件扫描路径
* 老版本或 Dubbox，使用：`<dubbo:annotation package="com.alibaba.dubbo.test.service" /> `

## Java Config 配置

* 使用 `@Configuration` 注解开启 Java Config 并使用 `@Bean` 进行公共模块的 bean 配置，参考：[API配置](http://dubbo.apache.org/zh-cn/docs/user/configuration/api.html)。


## 自动化配置

* 最后开启 `@EnableDubboConfig`

# 参考

《[在 Dubbo 中使用注解](http://dubbo.apache.org/zh-cn/blog/dubbo-annotation.html)》

https://www.oschina.net/news/92687/dubbo-spring-boot-starter-1-0-0

https://mvnrepository.com/artifact/com.alibaba.boot/dubbo-spring-boot-starter

https://mvnrepository.com/artifact/com.alibaba/dubbo

https://github.com/apache/incubator-dubbo
