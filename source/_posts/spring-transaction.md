---
title: Spring 事务管理总结
date: 2019-01-31 22:52:17
updated:
tags: Java
---

Spring 事务管理方式有两种：

* 编程式事务管理
* 声明式事务管理
  * 基于 AspectJ 的 XML 方式
  * 基于 `@Transactional` 的注解方式

这里重点介绍下最常用的基于 `@Transactional` 的注解方式，其提供的配置属性如下：

![@Transactional](/img/spring/spring_annotation_transactional.png)

其中事务的传播行为需要留意下，是 Spring 特有的概念，与数据库无关。它是为了解决业务层方法之间互相调用的事务问题而引入的。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。有以下几种方式：

| 传播行为        | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `REQUIRED`      | 支持当前事务，如果不存在，就新建一个。默认配置。             |
| `SUPPORTS`      | ~~支持当前事务，如果不存在，就以非事务方式执行。~~           |
| `MANDATORY`     | ~~支持当前事务，如果不存在，就抛出异常。~~                   |
| `REQUIRES_NEW`  | 如果当前存在事务，挂起当前事务，创建一个新事务。             |
| `NOT_SUPPORTED` | ~~以非事务方式执行，如果当前存在事务，则挂起当前事务。~~     |
| `NEVER`         | ~~以非事务方式执行，如果当前存在事务，则抛出异常。~~         |
| `NESTED`        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与 `REQUIRED` 类似的操作。 |

强烈不建议使用非事务方式执行，因此上述标注删除线的传播行为不建议使用。

# 参考

https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#transaction

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Propagation.html