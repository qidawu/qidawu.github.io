---
title: Jakarta EE 系列（三）Bean Validation 规范与实践总结
date: 2020-10-05 23:32:08
updated:
tags: [Java]
typora-root-url: ..
---

# Bean Validation 规范

Bean Validation 为 JavaBean 和方法验证定义了一组元数据模型和 API 规范，常用于后端数据的声明式校验。

## RoadMap

Bean Validation 规范最早在 Oracle Java EE 下维护。

2017 年 11 月，Oracle 将 Java EE 移交给 Eclipse 基金会。 2018 年 3 月 5 日，Eclipse 基金会宣布 Java EE (Enterprise Edition) 被更名为 Jakarta EE。因此 Bean Validation 规范经历了下面两个阶段：

### JavaEE Bean Validation

> Bean Validation 1.0 ([JSR 303](https://www.jcp.org/en/jsr/detail?id=303)) was the first version of Java's standard for object validation.
>
> It was released in 2009 and is part of [Java EE 6](https://docs.oracle.com/javaee/6/index.html).

```XML
<!-- https://mvnrepository.com/artifact/javax.validation/validation-api -->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.0.0.GA</version>
</dependency>
```



> Bean Validation 1.1 ([JSR 349](https://www.jcp.org/en/jsr/detail?id=349)) was finished in 2013. [Changes between Bean Validation 1.0 and 1.1](https://beanvalidation.org/1.1/)
>
> It is part of [Java EE 7](https://docs.oracle.com/javaee/7/index.html).

```XML
<!-- https://mvnrepository.com/artifact/javax.validation/validation-api -->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
```



> Bean Validation 2.0 ([JSR 380](https://www.jcp.org/en/jsr/detail?id=380)) was finished in August 2017. [Changes between Bean Validation 2.0 and 1.1](https://beanvalidation.org/2.0-jsr380/)
>
> It's part of [Java EE 8](https://www.oracle.com/java/technologies/java-ee-8.html) (but can of course be used with plain Java SE as the previous releases).

```XML
<!-- https://mvnrepository.com/artifact/javax.validation/validation-api -->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
```

### Jakarta Bean Validation

> [Jakarta Bean Validation 2.0](https://jakarta.ee/specifications/bean-validation/2.0/) was published in August 2019. There are no changes between Jakarta Bean Validation 2.0 and Bean Validation 2.0 except for the GAV: it is now `jakarta.validation:jakarta.validation-api`.
>
> It's part of [Jakarta EE 8](https://projects.eclipse.org/releases/jakarta-ee-8) (but can of course be used with plain Java SE as the previous releases).

```XML
<!-- https://mvnrepository.com/artifact/jakarta.validation/jakarta.validation-api -->
<dependency>
    <groupId>jakarta.validation</groupId>
    <artifactId>jakarta.validation-api</artifactId>
    <version>2.0.2</version>
</dependency>
```



> [Jakarta Bean Validation 3.0](https://jakarta.ee/specifications/bean-validation/3.0/) was released in Wednesday, October 7, 2020.
>
> This release is part of [Jakarta EE 9](https://projects.eclipse.org/releases/jakarta-ee-9).

```XML
<!-- https://mvnrepository.com/artifact/jakarta.validation/jakarta.validation-api -->
<dependency>
    <groupId>jakarta.validation</groupId>
    <artifactId>jakarta.validation-api</artifactId>
    <version>3.0.0</version>
</dependency>
```

## Constraints

Constraints 约束是 Bean Validation 规范的核心。约束是通过**约束注解**和一系列**约束验证实现**的组合来定义的。约束注解可应用于类型、字段、方法、构造函数、参数、容器元素或其它约束注解。

> 一个 constraint 通常由 annotation 和相应的 constraint validator 组成，它们是一对多的关系。也就是说可以有多个 constraint validator 对应一个 annotation。在运行时，Bean Validation 框架本身会根据被注释元素的类型来选择合适的 constraint validator 对数据进行验证。
>
> 有些时候，在用户的应用中需要一些更复杂的 constraint。Bean Validation 提供扩展 constraint 的机制。可以通过两种方法去实现，一种是组合现有的 constraint 来生成一个更复杂的 constraint，另外一种是开发一个全新的 constraint。

下表列出了常用 Constraints：

### 官方 Constraints

表 1. Bean Validation 1.x 中内置的 Constraints，更多官方 Constraints 详见官方新版规范：[Jakarta Bean Validation specification](https://jakarta.ee/specifications/bean-validation/3.0/jakarta-bean-validation-spec-3.0.html)

| **Constraint**                | **详细信息**                                             |
| :---------------------------- | :------------------------------------------------------- |
| `@Null`                       | 被注释的元素必须为 `null`                                |
| `@NotNull`                    | 被注释的元素必须不为 `null`                              |
| `@AssertTrue`                 | 被注释的元素必须为 `true`                                |
| `@AssertFalse`                | 被注释的元素必须为 `false`                               |
| `@Min(value)`                 | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| `@Max(value)`                 | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| `@DecimalMin(value)`          | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| `@DecimalMax(value)`          | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| `@Size(max, min)`             | 被注释的元素的大小必须在指定的范围内                     |
| `@Digits (integer, fraction)` | 被注释的元素必须是一个数字，其值必须在可接受的范围内     |
| `@Past`                       | 被注释的元素必须是一个过去的日期                         |
| `@Future`                     | 被注释的元素必须是一个将来的日期                         |
| `@Pattern(value)`             | 被注释的元素必须符合指定的正则表达式                     |

### 第三方 Constraints

表 2. Hibernate Validator 附加的 Constraints，更多 Constraints 详见：[Additional constraints](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-defineconstraints-hv-constraints)

| **Constraint** | **详细信息**                           |
| :------------- | :------------------------------------- |
| `@Email`       | 被注释的元素必须是电子邮箱地址         |
| `@Length`      | 被注释的字符串的大小必须在指定的范围内 |
| `@NotEmpty`    | 被注释的字符串的必须非空               |
| `@Range`       | 被注释的元素必须在合适的范围内         |

其它第三方 Constraints：

* [Java Bean Validation Extension](https://github.com/nomemory/java-bean-validation-extension) offers a set of useful constraints (`@Alphanumeric`, `@IPv6`, `@StartsWith`...).
* [Collection Validators](https://github.com/jirutka/validator-collection) allows to define constraints on elements of collections.
* ......

## 核心 API

核心类 [`javax.validation.Validation`](https://docs.oracle.com/javaee/7/api/javax/validation/Validation.html) 作为 Bean Validation 的入口点，提供了三种引导方式。下面代码演示了以最简单的方式创建默认的 [`ValidatorFactory`](https://docs.oracle.com/javaee/7/api/javax/validation/ValidatorFactory.html)，并获取 `Validator` 用以验证 Java Bean：

```java
import lombok.experimental.UtilityClass;

import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.ValidatorFactory;
import javax.validation.Validator;
import java.util.Set;

/**
 * JavaBean 校验器
 */
@UtilityClass
public class ValidatorUtil {

    private static final Validator VALIDATOR = Validation.buildDefaultValidatorFactory().getValidator();

    /**
     * 校验参数
     * @param obj 参数
     * @return 校验结果
     */
    public <T> Set<ConstraintViolation<T>> validate(T obj) {
        return VALIDATOR.validate(obj);
    }

}
```

涉及的 API 如下：

* 核心类 `javax.validation.Validation`：

  > Note:
  >
  > - The `ValidatorFactory` object built by the bootstrap process should be cached and shared amongst `Validator` consumers.
  > - This class is **thread-safe**.

* 接口 `javax.validation.ValidatorFactory`：

  > Factory returning initialized `Validator` instances.
  >
  > Implementations are **thread-safe** and instances are typically cached and reused.

* 接口 `javax.validation.Validator`：

  > Validates bean instances. Implementations of this interface must be **thread-safe**.

# 使用方式

## Hibernate Validator 实现

Hibernate 框架提供了各种子项目，如下：

![Hibernate Projects](/img/hibernate/hibernate_projects.png)

其中，子项目 Hibernate Validator 是 Bean Validation 规范的**官方认证实现**。

要在 Maven 项目中使用 Hibernate Validator，需要添加如下依赖项：

* 新版实现，由于该 artifact 已被移至此新 group，6.x 及以上新版建议优先使用：

  ```XML
  <!-- https://mvnrepository.com/artifact/org.hibernate.validator/hibernate-validator -->
  <dependency>
     <groupId>org.hibernate.validator</groupId>
     <artifactId>hibernate-validator</artifactId>
     <version>7.0.0.Final</version>
  </dependency>
  ```

* 遗留实现，可以下载到 5.x 及之前的老版本，例如：

  ```XML
  <!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-validator -->
  <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-validator</artifactId>
      <version>5.4.3.Final</version>
  </dependency>
  ```

Hibernate Validator 版本兼容性如下：

| Hibernate Validator                                        | Java    | Bean Validation 规范                                         | Expression Language (EL) 规范                                |
| ---------------------------------------------------------- | ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [7.0 series](http://hibernate.org/validator/releases/7.0/) | 8 or 11 | [Jakarta EE 9 - Bean Validation 3.0](https://jakarta.ee/specifications/bean-validation/3.0/) | [Jakarta EE 9 - Expression Language 4.0](https://jakarta.ee/specifications/expression-language/4.0/) |
| [6.1 series](http://hibernate.org/validator/releases/6.0/) | 8 or 11 | [Jakarta EE 8 - Bean Validation 2.0](https://jakarta.ee/specifications/bean-validation/2.0/) | [Jakarta EE 8 - Expression Language 3.0](https://jakarta.ee/specifications/expression-language/3.0/) |
| [5.0 series](http://hibernate.org/validator/releases/5.0/) | 6 or 7  | [JavaEE 7 - Bean Validation 1.1 (JSR 349)](https://beanvalidation.org/1.1/) | Java EE 7 - Expression Language 3.0 (JSR 341)                |

引入 Hibernate Validator 后，将传递依赖 Bean Validation API 规范相应的版本，无需重复引入：

> Hibernate's Jakarta Bean Validation reference implementation. This transitively pulls in the dependency to the Bean Validation API.

详见 [/org/hibernate](https://repo1.maven.org/maven2/org/hibernate/) 该子项目 parent pom.xml 的版本号定义

## Dubbo 参数验证

https://dubbo.apache.org/zh/docs/v2.7/user/examples/parameter-validation/

## Spring MVC 参数验证

[参考：Spring MVC 方法参数验证](/2016/06/25/spring-mvc/#方法参数验证)

# 常见问题

## 缺少依赖

```
HV000183: Unable to load 'javax.el.ExpressionFactory'. Check that you have the EL dependencies on the classpath, or use ParameterMessageInterpolator instead
```

原因：缺少 `Unified Expression Language (EL)` 规范的实现依赖，即 Glassfish。

如果查看 Hibernate Validator 的 pom.xml，会发现该依赖被声明为 `provided`（如下），表示该依赖在运行时由 Java EE container 容器提供，因此无须重复引入。而对于 Spring Boot 应用来说，则需要添加此依赖。

```XML
<dependency>
  <groupId>org.glassfish</groupId>
  <artifactId>javax.el</artifactId>
  <scope>provided</scope>
</dependency>
```

解决方案详见如下：

> Hibernate Validator also requires an implementation of the Unified Expression Language ([JSR 341](http://jcp.org/en/jsr/detail?id=341)) for evaluating dynamic expressions in constraint violation messages.
>
> When your application runs in a Java EE container such as [WildFly](http://wildfly.org/), an EL implementation is already provided by the container.
>
> In a Java SE environment, however, you have to add an implementation as dependency to your POM file. For instance, you can add the following dependency to use the JSR 341 reference implementation:
>
> ```
> <dependency>
>      <groupId>org.glassfish</groupId>
>      <artifactId>jakarta.el</artifactId>
>      <version>${version.jakarta.el-api}</version>
> </dependency>
> ```

注意，要使用与 Hibernate Validator 匹配的 `Unified Expression Language (EL)` 的依赖版本，详见《Hibernate Validator 版本兼容性》。

参考：https://stackoverflow.com/questions/24386771/javax-validation-validationexception-hv000183-unable-to-load-javax-el-express

# 参考

https://docs.oracle.com/javaee/7/index.html

https://docs.oracle.com/javaee/7/api/index.html

https://jakarta.ee/

https://jakarta.ee/specifications/bean-validation/

https://beanvalidation.org/

http://hibernate.org/validator/

[后台表单校验（JSR303）](https://blog.csdn.net/qq2413273056/article/details/84378194)