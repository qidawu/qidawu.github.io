---
title: Java 数据持久化系列（六）MyBatis 框架解析
date: 2018-03-24 18:43:34
updated:
tags: [Java, JDBC]
typora-root-url: ..
---

MyBatis 在 Java 业务开发领域可以说是首选的持久化框架，本文主要总结下 MyBatis 的核心配置和源码解析。

# MyBatis 产品组成

MyBatis 常用的产品组成汇总如下：

| Project Description                                          | Link                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| MyBatis SQL Mapper Framework for Java                        | https://github.com/mybatis/mybatis-3           |
| Integration with Spring                                      | https://github.com/mybatis/spring              |
| Integration with Spring Boot                                 | https://github.com/mybatis/spring-boot-starter |
| Code generator for MyBatis and iBATIS                        | https://github.com/mybatis/generator           |
| SQL Generator for MyBatis and Spring JDBC Templates          | https://github.com/mybatis/mybatis-dynamic-sql |
| ~~MyBatis Typehandlers JSR 310（日期时间API）<br/>(merged into mybatis core since 3.4.5)~~ | https://github.com/mybatis/typehandlers-jsr310 |

# MyBatis 核心架构

![](/img/java/mybatis/mybatis_core_architecture.png)

![](/img/java/mybatis/mybatis_core_architecture_2.png)

MyBatis 语句执行时的层次结构：

![](/img/java/mybatis/mybatis_core_architecture_3.jpg)

涉及的主要 API 如下：

![mybatis-api](/img/java/mybatis/mybatis-api.png)

## Executor

![mybatis_api_Executor](/img/java/mybatis/mybatis_api_Executor.png)

## StatementHandler

![mybatis_api_StatementHandler](/img/java/mybatis/mybatis_api_StatementHandler.png)

## ParameterHandler

![mybatis_api_ParameterHandler](/img/java/mybatis/mybatis_api_ParameterHandler.png)

## ResultSetHandler

![mybatis_api_ResultSetHandler](/img/java/mybatis/mybatis_api_ResultSetHandler.png)

## TypeHandler

![mybatis_api_TypeHandler](/img/java/mybatis/mybatis_api_TypeHandler.png)

# 使用方式

## 基于 Statement ID 的传统方式

![mybatis_statement_id](/img/java/mybatis/mybatis_statement_id.jpg)

## 基于 Mapper 接口的推荐方式

![mybatis_mapper](/img/java/mybatis/mybatis_mapper.jpg)

# 参考

http://www.mybatis.org/

https://github.com/mybatis

《MyBatis 从入门到精通》

《MyBatis 技术内幕》