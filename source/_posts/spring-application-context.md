---
title: Spring 应用上下文总结
date: 2017-06-01 22:35:15
updated:
tags: Java
---

`org.springframework.beans` 和 `org.springframework.context` 包为 Spring 框架 IoC 容器的提供基础。有两种形式的 Spring 容器：

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

# Bean Factory

[org.springframework.beans.factory.BeanFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html)

# Factory Bean

[org.springframework.beans.factory.FactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/FactoryBean.html) 用于在 IoC 容器中创建其它 bean，该接口定义如下：

```java
T getObject()  // Return an instance (possibly shared or independent) of the object managed by this factory.
Class<?> getObjectType()  // Return the type of object that this FactoryBean creates, or null if not known in advance.
boolean isSingleton()  // Is the object managed by this factory a singleton? That is, will getObject() always return the same object (a reference that can be cached)?
```

为什么要使用 FactoryBean？主要用于实现框架设施（framework facilities），例如：

* 当需要从 JNDI 查找对象（例如 `DataSource`）时，可以使用 `JndiObjectFactoryBean`。
* 当使用 Spring AOP 为 bean 创建代理时，可以使用 `ProxyFactoryBean`。
* 当需要在 IoC 容器中创建 Hibernate 的 `SessionFactory` 时，可以使用 `LocalSessionFactoryBean`。
* 当需要在 IoC 容器中创建 MyBatis 的 `SqlSessionFactory` 时，可以使用 `SqlSessionFactoryBean`。

大多数情况下，**用户并不需要自定义 FactoryBean**，因为 FactoryBean 通常是框架指定的，且无法在 Spring IoC 容器的范围之外使用。

# Bean 的生命周期

Spring Bean Factory 负责管理 bean 的生命周期。bean 的生命周期由回调方法组成，可以分为两类：

* **Post initialization** call back methods
* **Pre destruction** call back methods

![Spring Bean Life Cycle](/img/spring/spring-bean-life-cycle.png)

Spring 框架提供了以下四种控制 bean 生命周期事件的方法：

* 实现 Spring 框架的回调接口：
  * `org.springframework.beans.factory.InitializingBean#afterPropertiesSet()`
  * `org.springframework.beans.factory.DisposableBean#destroy()`
* 使用 JavaEE 规范 `javax.annotation` 包中提供的注解：
  - `@PostConstruct`
  - `@PreDestroy`
* 实现特定行为的 `*Aware` 接口
* 在 bean 配置文件中自定义 `init-method` 和 `destroy-method` 方法

## *Aware 接口解析

在日常的开发中，我们经常需要用到 Spring 容器本身的功能资源，可以通过 Spring 提供的一系列 `*Aware` 子接口来实现具体的功能：

![Aware 接口](/img/spring/aware_interface.png)

`*Aware` 是一个具有标识作用的超级接口，实现该接口的 bean 具有被 Spring 容器通知的能力，而被通知的方式就是通过回调，以依赖注入的方式为 bean 设置相应属性，这是一个典型的依赖注入的使用场景。

参考：[org.springframework.beans.factory.Aware](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/Aware.html)

# Bean 的作用域

使用 `@Scope` 注解定义 bean 的作用域，它可以与 `@Component` 或 `@Bean` 一起使用：

* 单例(Singleton):在整个应用中，只创建bean的一个实例。
* 原型(Prototype):每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。
* 会话(Session):在Web应用中，为每个会话创建一个bean实例。
* 请求(Rquest):在Web应用中，为每个请求创建一个bean实例。

# 参考

《[Spring in Action, 4th](https://www.manning.com/books/spring-in-action-fourth-edition)》

《[Spring BeanFactory和FactoryBean的区别](https://www.jianshu.com/p/05c909c9beb0)》

[org.springframework.beans.factory.BeanFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html)

[org.springframework.beans.factory.FactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/FactoryBean.html)

[ThreadPoolExecutorFactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolExecutorFactoryBean.html)

[LocalSessionFactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/orm/hibernate5/LocalSessionFactoryBean.html)

[SqlSessionFactoryBean](https://github.com/mybatis/spring/blob/master/src/main/java/org/mybatis/spring/SqlSessionFactoryBean.java)