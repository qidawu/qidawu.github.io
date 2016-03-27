---
title: "使用 AOP 和注解实现方法缓存"
date: 2016-03-27 15:48:21
updated: 
tags: Java
---

由于项目中散落着各种使用缓存的代码，这些缓存代码与业务逻辑代码交织耦合在一起既编写重复又难以维护，因此打算将这部分缓存代码抽取出来形成一个注解以便使用。

这样的需求最适合用 AOP 技术来解决了，来看看如何在 Spring 框架下使用 AOP 技术：

# 开启注解扫描

首先开启 Spring 注解扫描：

```xml
<context:component-scan base-package="your.package" />
```

以及开启 `@AspectJ` 切面注解扫描：

```xml
<aop:aspectj-autoproxy proxy-target-class="true" />
```

# 编写注解

然后使用 Java 语法编写一个注解：

```java
package your.package;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 方法级缓存
 * 标注了这个注解的方法返回值将会被缓存
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MethodCache {

    /**
     * 缓存过期时间，单位是秒
     */
    int expire();

}
```

# 切面（Aspect）

最后是编写一个切面。注：如果你对切面的概念已经很清楚，可以跳过本小结。

什么是切面？通俗来说就是“何时何地发生何事”，其组成如下：

> Aspect = Advice (what & when) + Pointcut (where)

其执行过程如下：

![An aspect’s functionality (advice) is woven into a program’s execution at one or more join points.](/img/java/aop_terminology.png)

## 通知（Advice）

通知（Advice）定义了**何时（when）**发生**何事（what）**。

Spring AOP 的切面（Aspect）可以搭配下面五种通知（Adive）注解使用：

| 通知              | 描述                                       |
| --------------- | ---------------------------------------- |
| @Before         | The advice functionality takes place before the advised method is invoked. |
| @After          | The advice functionality takes place after the advised method completes, regardless of the outcome. |
| @AfterReturning | The advice functionality takes place after the advised method successfully completes. |
| @AfterThrowing  | The advice functionality takes place after the advised method throws an exception. |
| @Around         | The advice wraps the advised method, providing some functionality before and after the advised method is invoked. |


## 切点（Pointcut）

切点（Pointcut）定义了切面在**何处（where）**执行。

Spring AOP 的切点（Pointcut）使用 AspectJ 的“切点表达式语言（Pointcut Expression Language）”进行定义。但要注意的是，Spring 仅支持其中一个子集：

![切面指示器（Aspectj Designator）](/img/java/aop_aspectj_designator.png)

切点表达式的语法如下：

![切点表达式（Pointcut Expression）](/img/java/aop_pointcut_expression.png)

# 完成切面

使用注解来创建切面，是 AspectJ 5 所引入的关键特性。在 AspectJ 5 之前，编写 AspectJ 切面需要学习一种 Java 语言的扩展，很不友好。在此我们使用注解来实现我们的切面：

```java
package your.package;

import java.io.Serializable;
import java.lang.annotation.Annotation;
import java.lang.reflect.Method;

import org.apache.commons.lang.StringUtils;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import com.kingdee.finance.cache.service.centralize.CentralizeCacheService;

/**
 * 方法级缓存拦截器
 */
@Aspect
@Component
public class MethodCacheInterceptor {

    private static final Logger logger = LoggerFactory.getLogger("METHOD_CACHE");
    private static final String CACHE_NAME = "Your unique cache name";

    @Autowired
    private CentralizeCacheService centralizeCacheService;

    /**
     * 搭配 AspectJ 指示器“@annotation()”可以使本切面成为某个注解的代理实现
     */
    @Around("@annotation(your.package.MethodCache)")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        String cacheKey = getCacheKey(joinPoint);
        Serializable serializable = centralizeCacheService.get(CACHE_NAME, cacheKey);
        if (serializable != null) {
            logger.info("cache hit，key [{}]", cacheKey);
            return serializable;
        } else {
            logger.info("cache miss，key [{}]", cacheKey);
            Object result = joinPoint.proceed(joinPoint.getArgs());
            if (result == null) {
                logger.error("fail to get data from source，key [{}]", cacheKey);
            } else {
                MethodCache methodCache = getAnnotation(joinPoint, MethodCache.class);
                centralizeCacheService.put(CACHE_NAME, methodCache.expire(), cacheKey, (Serializable) result);
            }
            return result;
        }
    }

    /**
     * 根据类名、方法名和参数值获取唯一的缓存键
     * @return 格式为 "包名.类名.方法名.参数类型.参数值"，类似 "your.package.SomeService.getById(int).123"
     */
    private String getCacheKey(ProceedingJoinPoint joinPoint) {
        return String.format("%s.%s", 
                 joinPoint.getSignature().toString().split("\\s")[1], StringUtils.join(joinPoint.getArgs(), ","));
    }

    private <T extends Annotation> T getAnnotation(ProceedingJoinPoint jp, Class<T> clazz) {
        MethodSignature sign = (MethodSignature) jp.getSignature();
        Method method = sign.getMethod();
        return method.getAnnotation(clazz);
    }

}
```

要注意的是，目前该实现存在两个限制：

1. 方法入参必须为基本数据类型或者字符串类型，使用其它引用类型的参数会导致缓存键构造有误；
2. 方法返回值必须实现  `Serializable` 接口；

# 投入使用

例如，使用本注解为一个“按 ID 查询列表”的方法加上五分钟的缓存：

```java
@MethodCache(expire = 300)
public List<String> listById(String id) {
    // return a string list.
}
```

# 总结

使用 AOP 技术，你可以在一个地方定义所有的通用逻辑，并通过**声明式（declaratively）**的方式进行使用，而不必修改各个业务类的实现。这种代码解耦技术使得我们的业务代码更纯粹、仅包含所需的业务逻辑。相比继承（inheritance）和委托（delegation），AOP 实现相同的功能，代码会更整洁。