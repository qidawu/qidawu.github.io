---
title: Spring Boot 自动配置及 Factories 机制总结
date: 2019-01-20 17:27:20
updated:
tags: Java
---

# 自动配置的原理

Spring Boot 应用中的"自动配置"是通过 `@EnableAutoConfiguration` 注解进行开启的。`@EnableAutoConfiguration` 可以帮助 Spring Boot 应用将所有符合条件的 `@Configuration` 配置类都加载到 Spring IoC 容器中。其背后使用了 Spring Factories 机制进行配置加载，即 spring-core 中的加载类 `SpringFactoriesLoader`。详见[文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/SpringFactoriesLoader.html)：

> SpringFactoriesLoader loads and instantiates factories of a given type from "META-INF/spring.factories" files which may be present in multiple JAR files in the classpath. The spring.factories file must be in Properties format, where the key is the fully qualified name of the interface or abstract class, and the value is a comma-separated list of implementation class names. 

Spring Boot 应用的自动配置流程如下：

![Spring SPI](/img/spring/enable-auto-configuration.png)

使用 `SpringFactoriesLoader` 加载指定 Auto Configure：

```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
......
```

结果如下：

![SpringFactoriesLoader 解析结果](/img/spring/SpringFactoriesLoader.png)

继续使用 `SpringFactoriesLoader` 解析加载 Auto Configuration Import Filters：

```
# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnClassCondition
......
```

针对这 118 个候选的 Auto Configure，执行 Import Filters 对应的条件注解：

* `OnClassCondition`

  ```java
  // 底层实现，加载成功则返回 true；加载失败则抛出 java.lang.ClassNotFoundException，捕获异常后返回 false
  private static Class<?> forName(String className, ClassLoader classLoader)
      throws ClassNotFoundException {
      if (classLoader != null) {
          return classLoader.loadClass(className);
      }
      return Class.forName(className);
  }
  ```

* `OnBeanCondition`

  ......

过滤结果如下：

![](/img/spring/condition.png)

`TRACE` 日志如下：

```
Filtered 30 auto configuration class in 1000 ms
```

# Spring Factories 机制

在日常工作中，我们可能需要实现一些 SDK 或者 Spring Boot Starter 给被人使用，这个时候我们就可以使用这个 Factories 机制。Factories 机制可以让 SDK 或者 Starter 的使用只需要很少甚至不需要进行配置，只需要在服务中引入我们的 jar 包即可完成自动加载及配置。

jar 包中 `META-INF/spring.factories` 配置如下：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.YourAutoConfig
```

加载 `YourAutoConfig` 配置类之后，会根据其条件化注解及传递依赖解析以实现 Spring 应用上下文里的 bean 自动配置，详见[《Spring Bean 条件化配置总结》](/2017/06/05/spring-conditional-bean/)。

本质上，Spring Factories 机制与 Java SPI 机制原理都差不多：都是通过指定的规则，在规则定义的文件中配置接口的各种实现，通过 key-value 的方式读取，并以反射的方式实例化指定的类型。

# 参考

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/SpringFactoriesLoader.html

https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/core/io/support/SpringFactoriesLoader.java

《[Spring Boot spring.factories vs @Enable annotations](https://stackoverflow.com/questions/42819558/spring-boot-spring-factories-vs-enable-annotations)》

https://www.cnblogs.com/zheting/p/6707035.html

http://www.cnblogs.com/whx7762/p/7832985.html