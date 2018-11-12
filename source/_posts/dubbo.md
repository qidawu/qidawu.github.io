---
title: Dubbo 框架小结
date: 2018-11-12 20:14:05
updated:
tags: Java
---

# 配置

## 配置之间的关系

![配置之间的关系](/img/dubbo/dubbo-config.jpg)

| XML 配置               | 注解配置     | 配置         | 解释                                                         |
| ---------------------- | ------------ | ------------ | ------------------------------------------------------------ |
| `<dubbo:application/>` |              | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者           |
| `<dubbo:registry/>`    |              | 注册中心配置 | 用于配置连接注册中心相关信息                                 |
| `<dubbo:monitor/>`     |              | 监控中心配置 | 用于配置连接监控中心相关信息，可选                           |
| `<dubbo:protocol/>`    |              | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方**被动接受** |
| `<dubbo:provider/>`    |              | 提供方配置   | 当 `ProtocolConfig` 和 `ServiceConfig` 某属性没有配置时，采用此缺省值，可选 |
| `<dubbo:service/>`     | `@Service`   | 服务配置     | 用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心 |
| `<dubbo:consumer/>`    |              | 消费方配置   | 当 `ReferenceConfig` 某属性没有配置时，采用此缺省值，可选    |
| `<dubbo:reference/>`   | `@Reference` | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心       |
| `<dubbo:method/>`      |              | 方法配置     | 用于 `ServiceConfig` 和 `ReferenceConfig` 指定方法级的配置信息 |
| `<dubbo:argument/>`    |              | 参数配置     | 用于指定方法参数配置                                         |
| `<dubbo:module/>`      |              | 模块配置     | 用于配置当前模块信息，可选                                   |

## 注解配置

如果想用现代的 Java Config 替代传统的 XML 配置方式，需要：

* 先使用 `@DubboComponentScan` 指定 dubbo 扫描路径；
* 然后使用 `@Configuration` 注解开启 Java Config 并使用 `@Bean` 进行以下公共模块的配置：
  * application
  * registry
  * monitor
  * protocol
  * module
  * provider
  * consumer

## 配置覆盖关系

配置覆盖关系：

* 方法级优先，接口级次之，全局配置再次之。
* 如果级别一样，则消费方优先，提供方次之。

规则二是指，所有配置最终都将转换为 URL 表示，并由服务提供方生成，经注册中心传递给消费方。URL 格式如下：`protocol://username:password@host:port/path?key=value&key=value`

参考[文档示例](http://dubbo.apache.org/zh-cn/docs/user/configuration/xml.html)。

## 配置分类

### 服务发现

### 服务治理

### 性能调优

# 参考

《[在 Dubbo 中使用注解](http://dubbo.apache.org/zh-cn/blog/dubbo-annotation.html)》