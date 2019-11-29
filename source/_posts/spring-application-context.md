---
title: Spring 应用上下文总结
date: 2017-06-01 22:35:15
updated:
tags: [Java, Spring]
typora-root-url: ..
---

`org.springframework.beans` 和 `org.springframework.context` 包为 Spring 框架 IoC 容器的提供基础。有两种形式的 Spring 容器：

* Bean Factory
* Application Context
  * spring-context 核心模块
    * `FileSystemXmlapplicationcontext`
    * `ClassPathXmlApplicationContext`
    * `AnnotationConfigApplicationContext`
  * spring-web 模块
    * `XmlWebApplicationContext`
    * `AnnotationConfigWebApplicationContext`

继承结构如下：

![BeanFactory](/img/spring/BeanFactory.png)

# Application Context

Spring 通过应用上下文（Application Context）装载 bean 的定义并将它们装配起来。Spring 应用上下文全权负责对象的创建、装配、配置它们并管理它们的整个生命周期。Spring 自带了多种应用上下文的实现，它们之间主要的区别仅仅在于如何加载配置：

## XML

- `FileSystemXmlapplicationcontext` 从文件系统下的一个或多个 XML 配置文件中加载上下文定义：

  ```java
  ApplicationContext ctx = new FileSystemXmlApplicationContext("c:/applicationContext.xml");
  ```

- `ClassPathXmlApplicationContext` 从类路径下的一个或多个 XML 配置文件中加载上下文定义：

  ```java
  ApplicationContext ctx = new ClassPathXmlApplicationContext("META-INF/spring/applicationContext.xml");
  ```

- `XmlWebApplicationContext` 从 Web 应用下的一个或多个 XML 配置文件中加载上下文定义，是 Web 应用程序使用的默认上下文类，因此不必在 `web.xml` 文件中显式指定这个上下文类。以下代码描述了 `web.xml` 中指向将由 `ContextLoaderListener` 监听器类载入的外部 XML 上下文文件的元素：

  ```xml
  <web-app>
      <context-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>/WEB-INF/applicationContext.xml</param-value>
      </context-param>
      <listener>
          <listener-class>
              org.springframework.web.context.ContextLoaderListener
          </listener-class>
      </listener>
      <servlet>
          <servlet-name>sampleServlet</servlet-name>
          <servlet-class>
              org.springframework.web.servlet.DispatcherServlet
          </servlet-class>
      </servlet>
   
  ...
  </web-app>
  ```

## Annotation

- `AnnotationConfigApplicationContext` 从一个或多个基于 Java 的配置类中加载 Spring 应用上下文：

  ```java
  // 使用构造函数来注册配置类 DubboApplication
  ApplicationContext ctx = new AnnotationConfigApplicationContext(org.apache.dubbo.config.DubboApplication.class);
  
  // 此外，还可以使用 register 方法来注册配置类 OtherApplication
  ctx.register(OtherApplication.class)
  ```

  ```java
  package org.apache.dubbo.config；
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  @Configuration
  public class DubboApplication {
      
      // 注册配置类将自动注册 @Bean 注解的方法名称返回的 bean
      @Bean
      public ApplicationConfig applicationConfig() {
          ApplicationConfig applicationConfig = new ApplicationConfig();
          applicationConfig.setName("dubbo-annotation-provider");
          return applicationConfig;
      }
  }
  ```

- `AnnotationConfigWebApplicationContext` 从一个或多个基于 Java 的配置类中加载 Spring Web 应用上下文，需要显示配置该类：

  ```xml
  <web-app>
      <context-param>
          <param-name>contextClass</param-name>
          <param-value>
              org.springframework.web.context.support.AnnotationConfigWebApplicationContext
          </param-value>
      </context-param>
      <context-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>
              demo.AppContext
          </param-value>
      </context-param>
      <listener>
          <listener-class>
              org.springframework.web.context.ContextLoaderListener
          </listener-class>
      </listener>
      <servlet>
          <servlet-name>sampleServlet</servlet-name>
          <servlet-class>
              org.springframework.web.servlet.DispatcherServlet
          </servlet-class>
          <init-param>
              <param-name>contextClass</param-name>
              <param-value>
                  org.springframework.web.context.support.AnnotationConfigWebApplicationContext
              </param-value>
          </init-param>
      </servlet>
   
  ...
  </web-app>
  ```

# Bean Factory

[org.springframework.beans.factory.BeanFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html)

# 参考

《[Spring in Action, 4th](https://www.manning.com/books/spring-in-action-fourth-edition)》

[org.springframework.beans.factory.BeanFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html)
