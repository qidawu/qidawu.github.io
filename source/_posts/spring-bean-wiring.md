---
title: Spring Bean 几种配置方式总结
date: 2016-06-04 22:26:34
updated:
tags: Java
---

Spring 提供几种配置方式，用于 bean 的声明及装配：

* 基于 XML 的显式配置
* 基于 Java 的显式配置（简称 JavaConfig）
* 自动化配置，即隐式的 bean 发现机制和自动装配

用户可以选择其中一种方式使用，也可以混搭使用。使用时的最佳实践如下：

* 建议尽可能地使用自动化配置的机制。显式配置越少越好，以避免显式配置所带来的维护成本。
* 当你必须要显式配置 bean 的时候（比如，有些源码不是由你来维护的，而当你需要为这些代码配置 bean 的时候），推荐使用类型安全并且比 XML 更加强大的 JavaConfig。
* 最后，只有当你想要使用便利的 XML 命名空间，并且在 JavaConfig 中没有同样的实现时，才应该使用 XML。

这里提供一个例子，其接口和实现类如下：

```java
// 接口
public interface CompactDisc { void play(); }

// 实现类
public class SgtPeppers implements CompactDisc {
    public void play() {
        System.out.println("Hello world!");
    }
}
```

# 自动化配置

Spring 从两个角度来实现 bean 的自动化配置：

- 组件扫描（component scanning）：Spring 会自动发现应用上下文中要创建的 bean。
- 自动装配（autowiring）：Spring 自动满足 bean 之间的依赖。

组件扫描（隐式的 bean 发现机制）和自动装配组合在一起能够发挥出强大的威力，它们能够将你的显式配置降低到最少。

## 声明组件

```java
// @Component 注解表明该类会作为组件类，并告知 Spring 要为这个类创建 bean。因此没有必要在 XML 或 Java Config 中显式配置该 bean。
@Component
public class SgtPeppers implements CompactDisc {
    public void play() {
        System.out.println("Hello world!");
    }
}
```

## 组件扫描

```java
// 显式声明 Java Config 配置类
@Configuration
// 组件扫描默认是不启用的。我们还需要显式配置一下 Spring，从而命令它去寻找带有 @Component 注解的类，并为其创建 bean。
// @ComponentScan 默认会扫描与配置类相同的包及其子包。有一个原因会促使我们明确地设置基础包，那就是我们想要将配置类放在单独的包中，使其与其他的应用代码区分开来。
@ComponentScan
public class CDPlayerConfig {
    // 没有显式地声明任何 bean，但由于开启了组件扫描，会在 Spring 容器中自动创建一个 SgtPeppers 类的 bean。
}
```

## 自动装配

为了测试组件扫描的功能，我们创建一个简单的 JUnit 单元测试。它会创建 Spring 上下文，并判断 bean 是否真的创建：

```java
@RunWith(SpringJUnit4ClassRunner.class) // 用于自动创建 Spring 的应用上下文
@ContextConfiguration(classes=CDPlayerConfig.class) // 指定要加载的配置
public class CDPlayerTest {
      
    // @Autowired 是 Spring 特有的注解，也可以使用 Java 依赖注入规范的 @Inject
    @Autowired
    private CompactDisc cd;

    @Test
    public void cdShouldNotBeNull() {
        assertNotNull(cd); // 测试通过
        cd.play(); // 输出 Hello world!
    }
      
}
```

如何处理自动装配的歧义性问题？有两种方案：

* 使用 `@Primary` 注解将可选 bean 中的某一个设为首选的 bean。`@Primary` 能够与 `@Component` 组合用在组件扫描的 bean 上，也可以与 `@Bean` 组合用在 Java 配置的 bean 声明中。 
* 使用限定符注解 `@Qualifier` 来帮助 Spring 将可选的 bean 的范围缩小到只有一个 bean。

# 基于 Java 的显式配置

尽管在很多场景下通过组件扫描和自动装配实现 Spring 的自动化配置是更为推荐的方式，但有时候自动化配置的方案行不通，因此需要显式配置 Spring。比如说，你想要将第三方库中的组件装配到你的应用中，在这种情况下，是没有办法在它的类上添加 `@Component` 和 `@Autowired` 注解的，因此就不能使用自动化装配的方案了。

在这种情况下，就必须要采用显式装配的方式。在进行显式配置的时候，有两种可选方案：Java 和 XML。Java Config 的优缺点如下：

- 优点：类型安全，对重构友好且不易出错。因为它就是 Java 代码，就像应用程序中的其它 Java 代码一样。
- 缺点：如果修改了 Java Config 类中的配置，就必须重新编译应用程序。

同时，Java Config 与其它的 Java 代码又有所区别，在概念上，它与应用程序中的业务逻辑和领域代码是不同的。尽管它与其它的组件一样都使用相同的语言进行表述，但 Java Config 是配置代码。这意味着它不应该包含任何业务逻辑，Java Config 也不应该侵入到业务逻辑代码之中。尽管不是必须的，但通常会将 Java Config **放到单独的包中**，使它与其他的应用程序逻辑分离开来，这样对于它的意图就不会产生困惑了。

下面是一个 Java Config 的例子：

```java
// 显式声明 Java Config 配置类
@Configuration
public class CDPlayerConfig {
    
    // 显式地声明 bean。@Bean 注解会告诉 Spring 这个方法将会返回一个对象，该对象要注册为 Spring 应用上下文中的 bean。方法体中包含了最终产生 bean 实例的逻辑。
    @Bean
    public CompactDisc getCompactDisc() { return new SgtPeppers(); }
    
    // 装配方式一：Spring 将会拦截所有对 getCompactDisc() 的调用，并确保直接返回该方法所创建的 bean，而不是每次都对其进行实际的调用。且默认情况下，Spring 中的 bean 都是单例的，因此多次调用只会返回同一个实例。
    @Bean
    public CDPlayer getCDPlayer1() {
        return new CDPlayer(getCompactDisc())；
    }
    
    // 装配方式二：上述通过调用方法来引用 bean 的方式有点令人困惑。而下面这种方式的好处是：
    // 1.不要求将 CompactDisc 声明到同一个配置类之中。
    // 2.不关注 Bean 的配置方式，你可以将配置分散到多个配置类、XML 文件以及自动扫描和装配 bean 之中，只要功能完整健全即可。
    @Bean
    public CDPlayer getCDPlayer2(CompactDisc cd) {
        return new CDPlayer(cd)；
    }
    
}
```

注意，这个 Java Config 需要放在 `@ComponentScan` 能够扫描到的路径之下，否则配置中所声明的 bean 将无法被 Spring 容器所注册。

## 条件化的 bean

假设你希望一个或多个 bean 只有在应用的类路径下包含特定的库时才创建。或者我们希望某个 bean 只有当另外某个特定的 bean 也声明了之后才会创建。我们还可能要求只有某个特定的环境变量设置之后，才会创建某个 bean。
在 Spring 4 之前，很难实现这种级别的条件化配置，但是 Spring 4 引入了一个新的 `@Conditional` 注解，它可以用到带有 `@Bean`注解的方法上。如果给定的条件计算结果为 `true`，就会创建这个 bean，否则的话，这个 bean 会被忽略。

详情参考另一篇博文：《[Spring Bean 条件化配置总结](/2016/06/05/spring-conditional-bean/)》

## @Import 与 @Enable* 注解

有时候我们需要引入一些外部的 Java Config 配置，这些配置往往是在其它 package 下。此时可以通过 `@Import` 注解导入这些外部 Java Config。

但这种导入声明有时不够直观，主流的做法是为其包装一层 `@Enable*` 注解，其字面意思是“开启某个功能”，非常直观。例如 Spring 框架所提供这类注解如下：

Spring Framework：

* spring-context
  * `@EnableAsync`
  * `@EnableScheduling`
  * `@EnableCaching`
  * `@EnableAspectJAutoProxy`
  * `@EnableLoadTimeWeaving`
  * `@EnableMBeanExport`
* spring-webmvc
  * `@EnableWebMvc`
* spring-webflux
  * `@EnableWebFlux`
* spring-websocket
  * `@EnableWebSocket`
  * `@EnableWebSocketMessageBroker`
* spring-tx
  * `@EnableTransactionManagement`
* spring-jms
  * `@EnableJms`

其它组件：

* spring-security
  * `@EnableWebSecurity`
* spring-data-jpa
  * `@EnableJpaRepositories`
* spring-boot-autoconfigure
  * `@EnableAutoConfiguration`

通过简单的 `@Enable*` 即可开启一项功能的支持，从而避免大量配置，大大降低使用难度。通过观察这些 `@Enable*` 注解的源码，可以发现所有的注解都有一个 `@Import` 注解，`@Import` 是用来导入配置类的，这也就意味着这些自动开启的实现其实就是导入了一些自动配置的 Bean。这些导入的配置主要分为以下三类：

1. 直接导入配置类
2. 依据条件选择配置类
3. 动态注册 Bean

# 基于 XML 的显式配置

XML 配置的缺点是比较复杂，且无法从编译期的类型检查中受益。除非是老项目维护，否则在新项目中已不再建议使用，此处不作过多介绍。

# 混合配置

在典型的 Spring 应用中，我们可能会同时使用自动化和显式配置。这些配置方案不是互斥的，可以将 Java Config 的组件扫描和自动装配和/或 XML 配置混合在一起：

```java
// 创建一个全局的根配置，并组合各种配置
@Configuration
// 通常会在根配置中启用组件扫描
@ComponentScan
// 导入 Java Config
@Import({FirstConfig.class, SecondConfig.class})
// 导入 XML 配置
@ImportResource("classpath:applicationContext.xml")
public class GlobalConfig() {}
```

# 参考

《[Spring in Action, 4th](https://www.manning.com/books/spring-in-action-fourth-edition)》

《[使用 Java 配置进行 Spring bean 管理](https://www.ibm.com/developerworks/cn/webservices/ws-springjava/)》

[Package org.springframework.context.annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/package-summary.html)

> Annotation support for the Application Context, including JSR-250 "common" annotations, component-scanning, and Java-based metadata for creating Spring-managed objects.

《[Spring4.x高级话题(六):@Enable*注解的工作原理](http://blog.longjiazuo.com/archives/1366)》

《[How those Spring @Enable* Annotations work](http://blog.fawnanddoug.com/2012/08/how-those-spring-enable-annotations-work.html)》