---
title: Spring 依赖注入总结
date: 2017-05-29 22:30:34
updated:
tags: [Java, Spring]
typora-root-url: ..
---

# 什么是依赖注入？

按照传统的做法，每个对象负责管理与自己相互协作的对象（即它所依赖的对象）的引用，这将会导致高度耦合和难以测试的代码：

```
             +---+
   +---new--->Bar|
   |         +---+
 +-+-+
 |Foo|
 +-+-+
   |         +---+
   +---new--->Baz|
             +---+
```

通过依赖注入（Dependency Injection），对象的依赖关系将由系统中负责协调各对象的**第三方组件**在创建对象的时候进行设定。对象无需自行创建或管理它们的依赖关系，依赖关系将被自动注入到需要它们的对象当中去，即做到“控制反转（IoC）”：

```
             +---+
  +--inject--+Bar|
  |          +---+
+-v-+
|Foo|
+-^-+
  |          +---+
  +--inject--+Baz|
             +---+
```

如果一个对象只通过接口（而不是具体实现或初始化过程）来表明依赖关系，那么这种依赖就能够在对象本身毫不知情的情况下，用不同的具体实现进行替换。这就是依赖注入所带来的最大收益——松耦合。

对依赖进行替换的一个最常用方法就是在测试的时候使用 mock 实现。

# 如何实现？

在面向对象的编程中，有几种实现控制反转的基本技术：

* 使用工厂模式（factory pattern）
* 使用服务定位模式（service locator pattern）
* 使用以下任何给定类型的**依赖注入（DI）**：
  * 构造方法注入（a constructor injection）
  * setter 方法注入（a setter injection）
  * 接口注入（an interface injection）

# 装配 Bean

创建应用对象之间协作关系的行为通常称为装配（wiring），这也是依赖注入（DI）的本质。

Spring 通过它的配置，能够了解这些组成部分是如何装配起来的。这样的话，就可以在不改变所依赖的类的情况下，修改依赖关系。

Spring 提供几种配置方式，用于 bean 的声明及装配，详见《[Spring Bean 几种配置方式总结](/2017/06/04/spring-bean-wiring/)》。

# 例子

## 例子一

通过依赖注入获取 Bean：

```java
// Spring 会将 service 对象作为集合注入到 list
@Autowired
private List<DemoService> demoServices;

// Spring 会将 service 的名字作为 key，service 对象作为 value 注入到 Map
@Autowired
private Map<String, DemoService> demoServiceMap;
```

## 例子二

通过 `BeanFactory` 主动获取 Bean：

![](/img/spring/getBean.png)

```java
@Service
public class XxxHandlerFactoryBean implements ApplicationContextAware {

    private final Map<String, Class<?>> map = new HashMap<>();

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        // 通过注解获取 Beans
        Map<String, Object> beans = applicationContext.getBeansWithAnnotation(PayMethod.class);
        beans.forEach((beanName, bean) -> {
            Class<?> beanClass = bean.getClass();
            PayMethod payMethod = beanClass.getAnnotation(PayMethod.class);
            map.put(payMethod.code(), beanClass);
        });
    }

    public Class<?> getBeanClass(String code) {
        Class<?> aClass = map.get(code);
        if (aClass == null) {
            throw new NotImplementedException("Not Implemented");
        }
        return aClass;
    }

}
```

## 例子三

下例通过 Collection Injection 实现自定义策略模式。

首先，创建策略注解。注意，此处还使用了 `@Service`，表示标注了 `@PayMethod` 注解的类都由 Spring Bean Factory 来创建对象并管理：

```java
import lombok.Getter;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.lang.annotation.*;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Service
public @interface PayMethod {
  
    PayMethodEnum code();

    @RequiredArgsConstructor
    @Getter
    enum PayMethodEnum {
       ...
    }

}
```

然后，创建策略类：

```java
/**
 * 策略接口
 */
public interface XxxHandler {
  ...
}

/**
 * 策略实现
 */
@PayMethod(code = PayMethod.PayMethodEnum.Aaa)
public class AaaHandler implements XxxHandler {
  ...
}
```

最后，创建工厂类：

```java
import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

import org.springframework.beans.factory.annotation.Autowired;
import org.apache.commons.lang3.NotImplementedException;

@Service
public class XxxHandlerFactoryBean {

    private Map<PayMethod.PayMethodEnum, XxxHandler> map;

    /**
     * 通过构造方法注入策略实现
     */
    @Autowired
    public XxxHandlerFactory(List<XxxHandler> handlers) {
        this.map = handlers.stream()
                .filter(handler -> getAnnotation(handler) != null)
                .collect(Collectors.toMap(handler -> getAnnotation(handler).code(), Function.identity()));
    }

    private PayMethod getAnnotation(XxxHandler handler) {
        return handler.getClass().getAnnotation(PayMethod.class);
    }

    public XxxHandler getHandler(String code) {
        PayMethod.PayMethodEnum payMethodEnum = PayMethod.PayMethodEnum.valueOfCode(code);
        XxxHandler handler = map.get(payMethodEnum);
        if (handler == null) {
            throw new NotImplementedException("Not Implemented");
        }
        return handler;
    }

}
```

# 参考

《[Spring in Action, 4th](https://www.manning.com/books/spring-in-action-fourth-edition)》

《[Dependency Injection, Design patterns using Spring and Guice](https://www.manning.com/books/dependency-injection)》

《[IoC模式（依赖、依赖倒置、依赖注入、控制反转）](https://www.cnblogs.com/fuchongjundream/p/3873073.html)》
