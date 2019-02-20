---
title: Spring Bean 条件化配置总结
date: 2017-06-05 23:29:06
updated:
tags: Java
---

# Spring 4.0 的条件化注解

> 假设你希望一个或多个 bean 只有在应用的类路径下包含特定的库时才创建。或者我们希望某个 bean 只有当另外某个特定的 bean 也声明了之后才会创建。我们还可能要求只有某个特定的环境变量设置之后，才会创建某个 bean。
> 在 Spring 4 之前，很难实现这种级别的条件化配置，但是 Spring 4 引入了一个新的 `@Conditional` 注解，它可以用到带有 `@Bean` 注解的方法上。如果给定的条件计算结果为 `true`，就会创建这个 bean，否则的话，这个 bean 会被忽略。

[@Conditional 注解](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Conditional.html)的源码如下：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```

[Condition 接口](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Condition.html)的源码如下：

```java
public interface Condition {
    boolean matches(ConditionContext var1, AnnotatedTypeMetadata var2);
}
```

其中，通过 `Condition` 接口的入参 `ConditionContext`，我们可以做到如下几点：

* 借助 `getRegistry()` 返回的 `BeanDefinitionRegistry` 检查 bean 定义；
* 借助 `getBeanFactory()` 返回的 `ConfigurableListableBeanFactory` 检查 bean 是否存在，甚至探查 bean 的属性；
* 借助 `getEnvironment()` 返回的 `Environment` 检查环境变量是否存在以及它的值是什么；
* 读取并探查 `getResourceLoader()` 返回的 `ResourceLoader` 所加载的资源；
* 借助 `getClassLoader()` 返回的 `ClassLoader` 加载并检查类是否存在。

## 环境与 profile

在开发软件的时候，有一个很大的挑战就是将应用程序从一个环境迁移到另外一个环境。开发阶段中，某些环境相关做法可能并不适合迁移到生产环境中，甚至即便迁移过去也无法正常工作。跨环境部署时会发生变化的几个典型例子：

- 数据库配置
- 加密算法
- 与外部系统的集成

解决办法：

- 在单独的 Java Config（或 XML）中配置每个 bean，然后在**构建时**根据不同的环境分别打包，典型方法是采用 Maven profile。这种方式的问题在于要为每种环境重新构建应用。
- **运行时**指定不同的环境变量，典型方法是采用 Spring profile bean。这种方式的好处在于无需为每种环境重新构建应用。

Spring profile bean 的使用方式：

```java
@Configuration
@Profile("dev") // 类级别（Spring 3.1 引入）
public class DevProfileConfig() {
    
}

@Configuration
public class GlobalConfig() {
    
    @Bean
    @Profile("dev") // 方法级别（Spring 3.2 引入）
    public DataSource embeddedDataSource() {
        ...
    }
    
    @Bean
    @Profile("prod") // 方法级别（Spring 3.2 引入）
    public DataSource jndiDataSource() {
        ...
    }
    
}
```

激活方式，使用属性：`spring.profiles.active` 和 `spring.profiles.default`。有多种方式来设置这两个属性：

* 作为 `DispatcherServlet` 的初始化参数；
* 作为 Web 应用的上下文参数；
* 作为 JNDI 条目；
* 作为环境变量；
* 作为 JVM 的系统属性；
* 在集成测试类上，使用 `@ActiveProfiles` 注解设置。

注意，Spring profile bean 注解 `@Profile` 底层其实也是基于 `@Conditional` 和 `Condition` 实现：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional({ProfileCondition.class})
public @interface Profile {
    String[] value();
}
```

# Spring Boot 的条件化注解

Spring Boot 没有引入任何形式的代码生成，而是利用了 Spring 4 的条件化 bean 配置特性，以及 Maven 和 Gradle 提供的传递依赖解析，以此实现 Spring 应用上下文里的**自动配置**。Spring Boot 实现的条件化注解如下：

![Spring Boot 实现的条件化注解](/img/spring/conditional_annotation.png)

Spring Boot 的 `@Conditional` 注解实现及相关支持类，详见文档：[Package org.springframework.boot.autoconfigure.condition](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/package-summary.html)

依赖配置：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
```

通过运行 `DEBUG=true mvn spring-boot:run`，可以看到 DEBUG 级别的日志输出，从而观察自动配置的详情（AUTO-CONFIGURATION REPORT）：

* Positive matches
* Negative matches
* Exclusions
* Unconditional classes

```
============================
CONDITIONS EVALUATION REPORT
============================

Positive matches:
-----------------

   GenericCacheConfiguration matched:
      - Cache org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration automatic cache type (CacheCondition)

   JmxAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.jmx.export.MBeanExporter' (OnClassCondition)
      - @ConditionalOnProperty (spring.jmx.enabled=true) matched (OnPropertyCondition)

   PropertyPlaceholderAutoConfiguration#propertySourcesPlaceholderConfigurer matched:
      - @ConditionalOnMissingBean (types: org.springframework.context.support.PropertySourcesPlaceholderConfigurer; SearchStrategy: current) did not find any beans (OnBeanCondition)


Negative matches:
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'org.aspectj.lang.annotation.Aspect' (OnClassCondition)

Exclusions:
-----------

    None

Unconditional classes:
----------------------

    org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration

    org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration

    org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration

```

# 参考

《[Spring in Action, 4th](https://www.manning.com/books/spring-in-action-fourth-edition)》

《[SpringBoot源码分析之条件注解的底层实现](https://www.jianshu.com/p/c4df7be75d6e)》