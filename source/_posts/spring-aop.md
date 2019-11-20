---
title: Spring AOP 面向切面编程总结
date: 2017-11-10 22:48:21
updated: 
tags: [Java, Spring]
---

# AOP 概念

前面我们重点关注了如何使用依赖注入（DI）管理和配置我们的应用对象，从而实现应用对象之间的解耦，而 AOP 主要实现“横切关注点（cross-cutting concern）”与它们所影响的对象之间的解耦。

在软件开发中，散布于应用中多处的功能被称为“横切关注点（cross-cutting concern）”。通常来讲，这些横切关注点从概念上是与应用的业务逻辑相分离的（但是往往会直接嵌入到应用的业务逻辑之中）。把这些横切关注点与业务逻辑相分离正是面向切面编程（AOP）所要解决的问题。

AOP 补充了 OOP 编程，通过提供另一种思考软件结构的方法。OOP 编程中的模块单元是“类（Class）”，而 AOP 编程中的模块单元是“切面（Aspect）”。

切面提供了取代继承和委托的另一种可选方案，而且在很多场景下更清晰简洁。在使用面向切面编程时，我们仍然在一个地方定义通用功能，但是可以通过**声明的方式**定义这个功能要以何种方式在何处应用，而无需修改受影响的类。横切关注点可以被模块化为特殊的类，这些类被称为切面（aspect）。这样做有两个好处：

- 首先，现在每个关注点都集中于一个地方，而不是分散到多处代码中；
- 其次，服务模块更简洁，因为它们只包含主要关注点（或核心功能）的代码，而次要关注点的代码被转移到切面中了。

AOP 的一些场景如下：

- 日志
- 事务，如 Spring Transactional
- 安全，如 Spring Security
- 缓存，如 Spring Cache

AOP 的知识点总结：

![AOP 总览](/img/spring/AOP.png)

# AOP 术语

与大多数技术一样，AOP 已经形成了自己的术语。下图展示了这些概念是如何关联在一起的：

![An aspect's functionality (advice) is woven into a program's execution at one or more join points.](/img/java/aop_terminology.png)

## 切面（Aspect）

什么是切面？通俗来说就是“何时何地发生何事”，其组成如下：

> Aspect = Advice (what & when) + Pointcut (where)

## 通知（Advice）

通知（Advice）定义了**何时（when）**发生**何事（what）**。

Spring AOP 的切面（Aspect）可以搭配下面五种通知（Advice）注解使用：

| 通知              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `@Before`         | The advice functionality takes place before the advised method is invoked. |
| `@After`          | The advice functionality takes place after the advised method completes, regardless of the outcome. |
| `@AfterReturning` | The advice functionality takes place after the advised method successfully completes. |
| `@AfterThrowing`  | The advice functionality takes place after the advised method throws an exception. |
| `@Around`         | The advice wraps the advised method, providing some functionality before and after the advised method is invoked. |


## 切点（Pointcut）

切点（Pointcut）定义了切面在**何处（where）**执行。

Spring AOP 的切点（Pointcut）使用 AspectJ 的“**切点表达式语言（Pointcut Expression Language）**”进行定义。但要注意的是，Spring 仅支持其中一个子集：

![切面指示器（Aspectj Designator）](/img/java/aop_aspectj_designator.png)

切点表达式的语法如下：

![切点表达式（Pointcut Expression）](/img/java/aop_pointcut_expression.png)

## 连接点（Join point）

连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

## 织入（Weaving）

织入是把切面应用到目标对象并创建**新的代理对象**的过程。切面（aspect）在指定的连接点（join point）被织入（weaving）到目标对象中。在目标对象的生命周期里有多个点可以进行织入：

| 生命周期 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 编译期   | 切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ 的织入编译器就是以这种方式织入切面的。 |
| 类加载期 | 切面在目标类加载到 JVM 时被织入。这种方式需要特殊的类加载器（Class Loader），它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5 的[加载时织入（load-time weaving, LTW）](http://www.eclipse.org/aspectj/doc/next/devguide/ltw.html)就支持以这种方式织入切面。 |
| 运行期   | 切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP 容器会为目标对象动态地创建一个代理对象。Spring AOP 就是以这种方式织入切面的。Spring AOP 构建在动态代理基础之上，因此，Spring 对 AOP 的支持局限于方法拦截。如果你的 AOP 需求超过了简单的方法调用（如构造器或属性拦截），那么你需要考虑使用 AspectJ 来实现切面。 |

# Spring AOP 与 AspectJ AOP 对比

这里总结下 Spring AOP 和 AspectJ AOP 两种织入方式的优缺点：

## Spring AOP 优点

* 使用方式比 AspectJ 简单，无需特殊的 LTW 或 AspectJ Compiler，仅在运行时通知对象

  > 通过在代理类中包裹切面，Spring 在运行期把切面织入到 Spring 管理的 bean 中。如下图所示，代理类封装了目标类，并拦截被通知方法的调用，再把调用转发给真正的目标 bean。当代理拦截到方法调用时，在调用目标 bean 方法之前，会执行切面逻辑。

  ![Spring 的切面由包裹了目标对象的代理类实现。代理类处理方法的调用，执行额外的切面逻辑，并调用目标方法](/img/spring/spring_aop_proxy.png)

  > 直到应用需要被代理的 bean 时，Spring 才创建代理对象。如果使用的是 `ApplicationContext` 的话，在 `ApplicationContext` 从 `BeanFactory` 中**加载所有 bean 的时候**，Spring 才会创建被代理的对象。因为 Spring 运行时才创建代理对象，所以我们不需要特殊的编译器来织入 Spring AOP 的切面。

* Advice 使用 Java 编写，使用成本低

  > Spring 所创建的通知（Advice）都是用标准的 Java 类编写的。这样的话，我们就可以使用与普通 Java 开发一样的集成开发环境（IDE）来开发切面。而且，定义通知所应用的切点通常会使用注解或在 Spring 配置文件里采用 XML 来编写，这两种语法对于Java开发者来说都是相当熟悉的。
  >
  > AspectJ 与之相反。虽然 AspectJ 现在支持基于注解的切面，但 AspectJ 最初是以 Java 语言扩展的方式实现的。这种方式有优点也有缺点。通过特有的 AOP 语言，我们可以获得更强大和细粒度的控制，以及更丰富的 AOP 工具集，但是我们需要额外学习新的工具和语法。

## Spring AOP 缺点

* 只支持方法级别的 join points，局限于 public 方法拦截。

  > 正如前面所探讨过的，通过使用各种 AOP 方案可以支持多种连接点模型。因为 Spring 基于**动态代理**，所以 Spring 只支持**方法连接点**。这与一些其他的 AOP 框架是不同的，例如 AspectJ 和 JBoss，除了方法切点，它们还提供了字段和构造器接入点。Spring 缺少对字段连接点的支持，无法让我们创建细粒度的通知，例如拦截对象字段的修改。而且它不支持构造器连接点，我们就无法在 bean 创建时应用通知。
  >
  > 但是方法拦截可以满足绝大部分的需求。如果需要方法拦截之外的连接点拦截功能，那么我们可以利用 Aspect 来补充 Spring AOP 的功能。

* 受限于 JDK Proxy 以及 CGLib Proxy**（Spring 风格）**的特点，不支持方法自调用（self-invocation），即同一个类中的方法调用无法应用切面。

* 无法将切面应用到非 Spring 工厂创建的 bean。

* 有一定的运行时开销。

## AspectJ AOP 优点

* 支持所有类型的 join points（构造器、字段、方法），可以做细粒度的控制。
* 支持任意访问修饰符（如 protected、private）、支持方法自调用（self-invocation）。

## AspectJ AOP 缺点

* 使用上要小心，确保切面只织入到需要被织入的地方。
* 需要额外的 LTW 或 AspectJ Compiler。

# Spring 对 AOP 的支持

Spring AOP 的设计理念和大多数其它 AOP 框架不同。目标并不是为了提供一个最完整的 AOP 实现，而是为了提供一个 AOP 实现与 Spring IoC 的紧密集成，以帮助解决企业级应用的常见问题。

## Spring AOP 的两种实现方式

Spring AOP 支持两种模式的动态代理，JDK Proxy 或者 CGLib：

![Spring AOP process](/img/spring/springaop-process.png)

两种模式的优势如下：

- JDK Proxy

  - 最小化依赖关系，减少依赖意味着简化开发和维护，JDK 本身的支持，可能比 CGLib 更加可靠。
  - 平滑进行 JDK 版本升级，而字节码类库通常需要进行更新以保证在新版 Java 上能够使用。
  - 代码实现简单，主要利用 JDK 反射机制。

- CGLib Proxy

  - 有的时候调用目标可能不便实现额外接口，从某种角度看，限定调用者实现接口是有些侵入性的实践，类似 CGLib 动态代理就没有这种限制。CGLib 动态代理采取的是创建目标类的子类的方式，因为是子类化，我们可以达到近似使用被调用者本身的效果。
  - 只操作我们关心的类，而不必为其它相关类增加工作量。
  - 性能更好。

核心源码解析：

![AopProxy 实现结构](/img/spring/AopProxy.png)

`org.springframework.aop.framework.DefaultAopProxyFactory` 工厂类负责判断创建哪个 `AopProxy` 实现：

```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

    @Override
    public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
        if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
            Class<?> targetClass = config.getTargetClass();
            if (targetClass == null) {
                throw new AopConfigException("TargetSource cannot determine target class: " +
                        "Either an interface or a target is required for proxy creation.");
            }
            if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
                return new JdkDynamicAopProxy(config);
            }
            return new ObjenesisCglibAopProxy(config);
        }
        else {
            return new JdkDynamicAopProxy(config);
        }
    }
    
    /**
     * Determine whether the supplied {@link AdvisedSupport} has only the
     * {@link org.springframework.aop.SpringProxy} interface specified
     * (or no proxy interfaces specified at all).
     */
    private boolean hasNoUserSuppliedProxyInterfaces(AdvisedSupport config) {
        Class<?>[] ifcs = config.getProxiedInterfaces();
        return (ifcs.length == 0 || (ifcs.length == 1 && SpringProxy.class.isAssignableFrom(ifcs[0])));
    }
    
}
```

`org.springframework.aop.framework.CglibAopProxy`

`org.springframework.aop.framework.JdkDynamicAopProxy`，两个关键方法：

```java
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {
    /**
     * 创建基于接口的 JDK 动态代理
     **/
    @Override
    public Object getProxy(ClassLoader classLoader) {
        if (logger.isDebugEnabled()) {
            logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
        }
        Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
        findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
        return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
    }
    
    /**
     * Implementation of {@code InvocationHandler.invoke}.
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        ...
        // Get the interception chain for this method.
        List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
        
        ...
        // We need to create a method invocation...
        invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
        // Proceed to the joinpoint through the interceptor chain.
        retVal = invocation.proceed();
        
        ...
    }
}
```

官方文档的一些关键摘录：

### AOP Proxies

Spring AOP defaults to using standard JDK dynamic proxies for AOP proxies. This enables any interface (or set of interfaces) to be proxied.

Spring AOP can also use CGLIB proxies. This is necessary to proxy classes rather than interfaces. By default, CGLIB is used if a business object does not implement an interface. As it is good practice to program to interfaces rather than classes, business classes normally implement one or more business interfaces. It is possible to [force the use of CGLIB](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-proxying), in those (hopefully rare) cases where you need to advise a method that is not declared on an interface or where you need to pass a proxied object to a method as a concrete type.

It is important to grasp the fact that Spring AOP is proxy-based. See [Understanding AOP Proxies](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-understanding-aop-proxies) for a thorough examination of exactly what this implementation detail actually means.

### Proxying Mechanisms

Spring AOP uses either JDK dynamic proxies or CGLIB to create the proxy for a given target object. (JDK dynamic proxies are preferred whenever you have a choice).

If the target object to be proxied implements at least one interface, a JDK dynamic proxy is used. All of the interfaces implemented by the target type are proxied. If the target object does not implement any interfaces, a CGLIB proxy is created.

If you want to force the use of CGLIB proxying (for example, to proxy every method defined for the target object, not only those implemented by its interfaces), you can do so. However, you should consider the following issues:

- `final` methods cannot be advised, as they cannot be overridden.
- As of Spring 3.2, it is no longer necessary to add CGLIB to your project classpath, as CGLIB classes are repackaged under `org.springframework` and included directly in the spring-core JAR. This means that CGLIB-based proxy support “just works”, in the same way that JDK dynamic proxies always have.
- As of Spring 4.0, the constructor of your proxied object is NOT called twice any more, since the CGLIB proxy instance is created through Objenesis. Only if your JVM does not allow for constructor bypassing, you might see double invocations and corresponding debug log entries from Spring’s AOP support.

To force the use of CGLIB proxies, set the value of the `proxy-target-class` attribute of the `<aop:config>` element to true, as follows:

```xml
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```

To force CGLIB proxying when you use the @AspectJ auto-proxy support, set the `proxy-target-class` attribute of the `<aop:aspectj-autoproxy>` element to `true`, as follows:

```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

## Spring AOP 切面声明的两种方式

Spring 2.0 之后提供了以下两种方式，为编写自定义切面引入了一种更简单和更强大的方式：

### schema-based approach

[schema-based approach](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-schema)，手工声明切面方式。下面是一个完整的例子：

```xml
<!-- this is the object that will be proxied by Spring's AOP infrastructure -->
<bean id="personService" class="x.y.service.DefaultPersonService"/>

<!-- this is the actual advice itself -->
<bean id="myAspect" class="x.y.MyAspect"/>

<!-- cglib 代理方式配置：proxy-target-class="true" -->
<aop:config>
    <aop:aspect ref="myAspect">
        <aop:pointcut id="theExecutionOfSomePersonServiceMethod" expression="execution(* x.y.service.PersonService.getPerson(String,int)) and args(name, age)"/>
        <aop:around pointcut-ref="theExecutionOfSomePersonServiceMethod" method="process"/>
    </aop:aspect>
</aop:config>
```

### @AspectJ annotation style

[@AspectJ annotation style](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj)

- `@AspectJ` 注解风格作为 AspectJ 5 发行版的一部分被引入。Spring 利用了这个注解，使用了 AspectJ 提供的类库进行切点解析和匹配。然而，AOP 运行时仍然是纯 Spring AOP，并且不依赖于 AspectJ 编译器和织入器。
- The `@AspectJ` support can be enabled with XML- or Java-style configuration. In either case, you also need to ensure that AspectJ’s `aspectjweaver.jar` library is on the classpath of your application (version 1.8 or later).

配置如下：

* Java Config 方式：

  ```java
  // 声明 Java Config
  @Configuration
  // 开启组件扫描，将 MethodCacheInterceptor 作为 bean 注册到 Spring 容器
  @ComponentScan
  // 开启自动代理
  @EnableAspectJAutoProxy
  public class ConertConfig {}
  ```

  注解 `@EnableAspectJAutoProxy` 用于开启 AspectJ 自动代理，为使用 `@Aspect` 注解的 bean 创建一个代理。其中 `proxyTargetClass` 属性用于控制代理方式：

  * true 表示开启 CGLIB 风格的子类继承代理（CGLIB-style 'subclass' proxy）
  * 默认为 false 表示开启基于接口的 JDK 动态代理（interface-based JDK proxy）

* XML 配置方式：

  ```xml
  <context:component-scan base-package="your.package" />
  <aop:aspectj-autoproxy />  <!-- 代理方式配置：proxy-target-class="true" -->
  ```

# 例子

由于项目中散落着各种使用缓存的代码，这些缓存代码与业务逻辑代码交织耦合在一起既编写重复又难以维护，因此打算将这部分缓存代码抽取出来形成一个注解以便使用。

这样的需求最适合通过 AOP 来解决了，来看看如何在 Spring 框架下通过 AOP 和注解实现方法缓存：

## 自定义注解

首先，自定义一个方法注解：

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

## 编写切面

使用注解来创建切面，是 AspectJ 5 所引入的关键特性。在 AspectJ 5 之前，编写 AspectJ 切面需要学习一种 Java 语言的扩展，很不友好。在此我们使用注解来实现我们的切面：

```java
package your.package;

import java.io.Serializable;
import java.lang.annotation.Annotation;
import java.lang.reflect.Method;

import lombok.extern.slf4j.Slf4j;

import org.apache.commons.lang.StringUtils;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import com.kingdee.finance.cache.service.centralize.CentralizeCacheService;

/**
 * 方法级缓存拦截器
 */
@Aspect
@Component
@Slf4j
public class MethodCacheInterceptor {

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
            log.info("cache hit，key [{}]", cacheKey);
            return serializable;
        } else {
            log.info("cache miss，key [{}]", cacheKey);
            Object result = joinPoint.proceed(joinPoint.getArgs());
            if (result == null) {
                log.error("fail to get data from source，key [{}]", cacheKey);
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

## 开启动态代理

最后，开启 Spring 的组件扫描、自动代理功能：

```java
// 声明 Java Config
@Configuration
// 开启组件扫描，将 MethodCacheInterceptor 作为 bean 注册到 Spring 容器
@ComponentScan
@EnableAspectJAutoProxy
public class ConertConfig {}
```

## 投入使用

例如，使用本注解为一个“按 ID 查询列表”的方法加上五分钟的缓存：

```java
@MethodCache(expire = 300)
public List<String> listById(String id) {
    // return a string list.
}
```

## 总结

使用 AOP 技术，你可以在一个地方定义所有的通用逻辑，并通过**声明式（declaratively）**的方式进行使用，而不必修改各个业务类的实现。这种代码解耦技术使得我们的业务代码更纯粹、仅包含所需的业务逻辑。相比继承（inheritance）和委托（delegation），AOP 实现相同的功能，代码会更整洁。

# 参考

《[Spring in Action, 4th](https://www.manning.com/books/spring-in-action-fourth-edition)》

https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop

《[AspectJ Docs](https://www.eclipse.org/aspectj/docs.php)》

《[AspectJ Compiler (ajc)](http://www.eclipse.org/aspectj/doc/released/devguide/ajc-ref.html)》

https://github.com/cglib/cglib/wiki

[@EnableAspectJAutoProxy](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/EnableAspectJAutoProxy.html)

https://www.cnblogs.com/xrq730/p/6661692.html