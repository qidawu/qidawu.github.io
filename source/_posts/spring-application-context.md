---
title: Spring 应用上下文总结
date: 2017-06-11 22:35:15
updated:
tags: Java
---

Spring 容器：

* Bean Factory

* Application Context

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

# Bean 的生命周期

# Bean 的作用域

使用 `@Scope` 注解定义 bean 的作用域，它可以与 `@Component` 或 `@Bean` 一起使用：

* 单例(Singleton):在整个应用中，只创建bean的一个实例。
* 原型(Prototype):每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。
* 会话(Session):在Web应用中，为每个会话创建一个bean实例。
* 请求(Rquest):在Web应用中，为每个请求创建一个bean实例。

# 参考

《[Spring in Action, 4th](https://www.manning.com/books/spring-in-action-fourth-edition)》