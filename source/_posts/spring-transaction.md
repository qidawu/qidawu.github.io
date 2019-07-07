---
title: Spring 事务管理总结
date: 2019-02-17 22:52:17
updated:
tags: Spring
---

本文梳理 Spring 事务管理的方方面面，总览如下：

![](/img/spring/transaction/transaction_management.png)

# Spring 框架的事务支持模型的优点

全面的事务支持是使用 Spring 框架的最有说服力的理由之一。Spring 框架为事务管理提供了一致的抽象层，并具有以下优势：

* 跨不同事务 API 的一致编程模型，如 JTA (Java Transaction API)、JPA (Java Persistence API)、JDBC、Hibernate、MyBatis。
* 支持[声明式事务管理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#transaction-declarative)，可通过 XML 或注解进行配置。
* 比复杂的事务 API（如 JTA）更简单的[编程式事务管理 API](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#transaction-programmatic)。
* 与 Spring 框架的数据访问抽象层集成。

# 声明式事务管理

## 理解声明式事务实现

关于 Spring 框架的声明式事务支持，最重要的概念是掌握其通过 [AOP 代理](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-understanding-aop-proxies)来启用此支持，并且事务 advice 由元数据（基于 XML 或注释 `@Transactional`）驱动。AOP 与事务元数据的组合产生 AOP 代理，该代理使用 `TransactionInterceptor` 搭配合适的 `PlatformTransactionManager` 实现来驱动围绕方法调用的事务代理。

`TransactionInterceptor` 的结构如下：

![TransactionInterceptor](/img/spring/transaction/TransactionInterceptor.png)

下图展示了调用事务代理方法的过程：

![事务代理调用](/img/spring/transaction/tx.png)

## 基于 XML 方式配置事务管理

使用 `<tx:advice/>` 创建事务 advice，并创建切面通过 `<aop:advisor/>` 指定该事务 advice 须应用到哪些切点之上：

```xml
<!-- the transactional advice (what 'happens'; see the <aop:advisor/> bean below) -->
<tx:advice id="txAdvice" transaction-manager="txManager">
    <!-- the transactional semantics... -->
    <tx:attributes>
        <!-- all methods starting with 'get' are read-only -->
        <tx:method name="get*" read-only="true"/>
        <!-- other methods use the default transaction settings (see below) -->
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>

<!-- ensure that the above transactional advice runs for any execution
        of an operation defined by the FooService interface -->
<aop:config>
    <aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
</aop:config>

......
```

事务配置可通过修改 `<tx:method/>` 的属性，详见脑图。

## 基于注解方式配置事务管理

除了使用基于 XML 的方式（`<tx:advice/>`）声明事务配置之外，您还可以使用基于注解的方式（`@Transactional` ）。直接在 Java 源代码中声明事务语义会使声明更靠近受影响的代码，易于配置和修改。这样之所以不存在过度耦合的原因是因为，无论如何，用于事务处理的代码几乎总是以事务的方式进行部署。

您可以将 `@Transactional` 注解应用于：

- 接口定义（interface）
- 接口上的方法
- 类定义（class）
- 类上的公有方法（public method on class）

`@Transactional` 提供的配置属性如下：

![@Transactional](/img/spring/transaction/spring_annotation_transactional.png)

### 开启事务支持

但是，仅仅使用 `@Transactional` 注解并不足以激活事务行为，还需要开启事务支持，可以使用以下方式：

* `<tx:annotation-driven/>`
* `@EnableTransactionManagement`

配置参数如下：

| XML 属性              | 注解属性                                                     | 默认                        | 描述                                                         |
| --------------------- | ------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| `transaction-manager` | N/A (see [`TransactionManagementConfigurer`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/TransactionManagementConfigurer.html) javadoc) | `transactionManager`        | 要使用的事务管理器的名称。仅在事务管理器的名称不是  `transactionManager` 时才需要设置。 |
| `mode`                | `mode`                                                       | `proxy`                     | 默认为代理模式（`proxy`），使用 Spring AOP 框架处理被 `@Transactional` 注解的 bean，仅适用于通过代理进入的方法调用。<br/>相反，替代模式（`aspectj`）使用Spring AspectJ 事务切面织入到受影响的类，修改目标类的字节码以应用于任何类型的方法调用（支持任意访问修饰符、支持自调用）。AspectJ 织入需要在类路径中包含 `spring-aspects.jar` 以及开启类加载期织入（load-time weaving）或编译期织入（compile-time weaving）。（参阅 [Spring 配置](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-aj-ltw-spring)） |
| `proxy-target-class`  | `proxyTargetClass`                                           | `false`                     | 仅适用于 `proxy` 模式。控制为使用 `@Transactional` 注解的类所创建的事务代理类型。如果 `proxy-target-class` 属性设置为 `true`，则创建基于类的代理（CGLib Proxy）。如果为 `false` 或者省略该属性，则创建基于标准 JDK 接口的代理（JDK Proxy）。（参阅[代理机制](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-proxying)） |
| `order`               | `order`                                                      | `Ordered.LOWEST_PRECEDENCE` | 参阅 [Advice 排序](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-ataspectj-advice-ordering)。 |

更多注意点，详见官方文档：

> The default advice mode for processing `@Transactional` annotations is `proxy`, which allows for interception of calls through the proxy only. Local calls within the same class cannot get intercepted that way. For a more advanced mode of interception, consider switching to `aspectj` mode in combination with compile-time or load-time weaving.

> In proxy mode (which is the default), only external method calls coming in through the proxy are intercepted. This means that self-invocation (in effect, a method within the target object calling another method of the target object) does not lead to an actual transaction at runtime even if the invoked method is marked with `@Transactional`.

> The `proxy-target-class` attribute controls what type of transactional proxies are created for classes annotated with the `@Transactional` annotation. If `proxy-target-class` is set to `true`, class-based proxies are created. If`proxy-target-class` is `false` or if the attribute is omitted, standard JDK interface-based proxies are created. (See [[aop-proxying\]](https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#aop-proxying) for a discussion of the different proxy types.)

> The Spring team recommends that you annotate only concrete classes (and methods of concrete classes) with the `@Transactional` annotation, as opposed to annotating interfaces. You certainly can place the `@Transactional` annotation on an interface (or an interface method), but this works only as you would expect it to if you use interface-based proxies. The fact that Java annotations are not inherited from interfaces means that, if you use class-based proxies (`proxy-target-class="true"`) or the weaving-based aspect (`mode="aspectj"`), the transaction settings are not recognized by the proxying and weaving infrastructure, and the object is not wrapped in a transactional proxy.

> When you use proxies, you should apply the `@Transactional` annotation only to methods with public visibility. If you do annotate protected, private or package-visible methods with the `@Transactional` annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. If you need to annotate non-public methods, consider using AspectJ.

`@EnableTransactionManagement` 注解主要用于导入 `TransactionManagementConfigurationSelector`，其判断 `mode` 属性：

* `mode` 为 `AdviceMode.PROXY`，返回配置  `org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration`，该 Java Config 用于配置以下 bean：
  * `TransactionInterceptor` 最关键的类
  * `TransactionAttributeSource`
  * `BeanFactoryTransactionAttributeSourceAdvisor`
* `mode` 为 `AdviceMode.ASPECTJ`，默认返回配置 `org.springframework.transaction.aspectj.AspectJTransactionManagementConfiguration`

## 事务的传播行为

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

在 Spring 管理的事务中，请注意物理事务和逻辑事务之间的区别，以及传播行为应用于两者之上时的区别。

### REQUIRED

![REQUIRED](/img/spring/transaction/tx_prop_required.png)

### REQUIRED_NEW

![REQUIRED_NEW](/img/spring/transaction/tx_prop_requires_new.png)

# 编程式事务管理

Spring 框架提供了两种编程式事务管理方法：

* 直接使用 Spring 框架最底层的 `PlatformTransactionManager`  的实现类；
* 更建议使用 Spring 框架封装过的 `TransactionTemplate` 事务模板类。

## 使用 `PlatformTransactionManager`

Spring 事务抽象的关键在于事务策略的概念。事务策略由`org.springframework.transaction.PlatformTransactionManager`接口定义 ，如下所示：

```java
public interface PlatformTransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

由于 `PlatformTransactionManager` 是一个接口，因此很容易按需 mock 或 stub。它与查找策略无关，例如JNDI。`PlatformTransactionManager` 的实现就像其它对象或 bean 一样在 Spring 框架的 IoC 容器中定义。仅此优势就让 Spring 框架事务成为一种有价值的抽象，即便是使用 JTA。与直接使用 JTA 相比，您可以更轻松地测试事务代码。

同时，为了与 Spring 的理念保持一致，`PlatformTransactionManager` 接口的所有方法抛出的  `TransactionException` 异常都是非受检的（即继承自 `java.lang.RuntimeException` 类）。事务基础设施故障几乎都是致命性的。只有极少数情况下，应用程序能够从事务故障中恢复过来。开发人员仍然可以选择 try catch `TransactionException`，但重点是开发人员不会**被迫**这样做。

`getTransaction(..)` 方法根据 `TransactionDefinition` 参数返回一个 `TransactionStatus` 对象 。`TransactionStatus` 表示一个新事务，但如果当前调用堆栈中存在匹配事务，则表示该已有事务，即 `TransactionStatus` 是与执行的线程相关联的。

`TransactionDefinition` 接口可以控制事务的传播行为、隔离级别、超时时间、只读状态，其结构如下：

![TransactionDefinition](/img/spring/transaction/TransactionDefinition.png)

`TransactionStatus` 接口为事务代码提供了一种简单的方法来控制事务执行和查询事务状态，其结构如下：

![TransactionDefinition](/img/spring/transaction/TransactionStatus.png)

在 Spring 中无论选择使用声明式还是编程式事务管理，定义正确的 `PlatformTransactionManager` 实现都是绝对必要的。Spring 提供了下面几种实现：

| 实现类                         | 包          | 工作环境        |
| ------------------------------ | ----------- | ---------------- |
| `DataSourceTransactionManager` | spring-jdbc | JDBC、Mybatis |
| `HibernateTransactionManager`  | spring-orm  | Hibernate        |
| `JpaTransactionManager`        | spring-orm  | JPA              |
| `JtaTransactionManager`        | spring-tx   | JTA              |

其继承关系如下：

![PlatformTransactionManager 实现](/img/spring/transaction/PlatformTransactionManager.png)

以最常用的 `DataSourceTransactionManager` 为例，重点看下都提供了哪些方法：

![DataSourceTransactionManager](/img/spring/transaction/DataSourceTransactionManager.png)

## 使用 `TransactionTemplate`

和 Spring 框架的其它模板类一样，`TransactionTemplate` 也采用了回调方法来减少样板代码。相比起直接使用 `PlatformTransactionManager` 接口，`TransactionTemplate` 可以让开发人员无须重复编写获取与释放事务资源的代码，从而更聚焦于业务代码。

![TransactionTemplate](/img/spring/transaction/TransactionTemplate.png)

你需要编写一个 `TransactionCallback` 实现（通常为匿名内部类），其中包含需要在事务上下文中执行的代码。然后传递给 `TransactionTemplate` 的 `execute(..)` 方法去执行：

![TransactionCallback](/img/spring/transaction/TransactionCallback.png)

由于 `TransactionTemplate` 继承自 `DefaultTransactionDefinition`，因此可以直接修改其属性进行事务配置（如传播行为、隔离级别、超时时间等）。`TransactionTemplate` 类的实例是线程安全的，实例并不维护任何会话状态，但是却会维护配置状态。因此，当多个类共享使用同一个 `TransactionTemplate` 类的实例时，如果其中一个需要使用不同的配置（例如不同的隔离级别），你需要创建两个不同的 `TransactionTemplate` 类的实例。

# 使用示例

```java
@Slf4j
@Service
public class TestDbService {
    
    @Autowired
    private TestDAO testDAO;
    @Autowired
    private PlatformTransactionManager transactionManager;
    @Autowired
    private TransactionTemplate transactionTemplate;

    /**
     * 要使用底层 `PlatformTransactionManager` 接口直接管理事务，请先注入所需的实现类。
     * 然后，通过 `TransactionDefinition` 和 `TransactionStatus` 对象启动、回滚和提交事务。
     */
    public Integer save() {
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        // explicitly setting the transaction name is something that can be done only programmatically
        def.setName("SomeTxName");
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        
        TransactionStatus status = transactionManager.getTransaction(def);
        try {
            Integer result = insert();
            transactionManager.commit(status);
            return result;
        } catch (Exception e) {
            log.error(e.getMessage(), e);
            transactionManager.rollback(status);
            return 0;
        }
    }

    /**
     * TransactionTemplate 采用了回调方法来减少样板代码。
     */
    public Integer save1() {
        return transactionTemplate.execute(status -> {
            try {
                return insert();
            } catch (Exception e) {
                log.error(e.getMessage(), e);
                status.setRollbackOnly();
                return 0;
            }
        });
    }

    /**
     * 使用注解方式配置事务，是最简单最推荐的方式。事务的参数配置详见脑图。
     */
    @Transactional(rollbackFor = Exception.class)
    public Integer save3() {
        return insert();
    }

}
```

# 参考

https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#transaction

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Propagation.html