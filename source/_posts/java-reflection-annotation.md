---
title: Java 反射篇（二）注解 API 总结
date: 2018-11-14 22:53:54
updated:
tags: Java
typora-root-url: ..
---

JDK 1.5 引入了注解，其主要用途如下：

* 生成文档，通过代码里标识的元数据生成 javadoc 文档。
* 编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
* 编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
* 运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例。

# Annotation

首先，注解也是一种 `class`，所有注解都默认继承自通用接口：`java.lang.annotation.Annotation`：

![annotation](/img/java/reflection/annotation.png)

从上图可见，JDK 提供的元注解（meta annotation）如下：

```java
@Native
@Documented
@Inherited
@Repeatable
@Target
@Retention
```

其中，`@Native`、`@Documented`、`@Inherited` 是一个“标记注解”，没有成员。

常用元注解的作用如下：

## @Inherited

`@Inherited` 用于定义**子类是否可继承父类定义的注解**。仅针对 `@Target(ElementType.TYPE)` 类型的注解有效，并且仅针对 `class` 的继承，对 `interface` 的继承无效。注意：

* 因此 `interface` 上标注的注解，无法被 JDK Proxy 动态代理生成的类所继承。
* 也因此 Spring AOP 的切点表达式 `@annotation(...)` 无法切到基于 JDK Proxy 的动态代理类。

https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Inherited.html

> Indicates that an annotation type is automatically inherited. If an Inherited meta-annotation is present on an annotation type declaration, and the user queries the annotation type on a class declaration, and the class declaration has no annotation for this type, **then the class's superclass will automatically be queried for the annotation type.** This process will be repeated until an annotation for this type is found, or the top of the class hierarchy (Object) is reached. If no superclass has an annotation for this type, then the query will indicate that the class in question has no such annotation.
>
> **Note that this meta-annotation type has no effect if the annotated type is used to annotate anything other than a class. Note also that this meta-annotation only causes annotations to be inherited from superclasses; annotations on implemented interfaces have no effect.**

## @Target

`@Target` 用于定义注解能够被标注于源码的哪些位置。可用字段参考 `ElementType` 枚举。

JDK 8 扩展了注解的上下文，现在注解几乎可以加到任何地方：局部变量、泛型类、⽗类与接⼝的实现，就连⽅法的异常也能添加注解。

JDK 8 引入的这两个枚举如下：

* `ElementType.TYPE_PARAMETER` 用于标注泛型的类型参数。
* `ElementType.TYPE_USE` 用于标注各种类型。

![extended_annotations_support](/img/java/reflection/extended_annotations_support.png)

## @Retention

`@Retention` 用于定义注解的生命周期。可用字段参考下面的 `RetentionPolicy` 枚举源码，常用的是 `RUNTIME` 类型：

```java
/**
* Annotation retention policy.  The constants of this enumerated type
* describe the various policies for retaining annotations.  They are used
* in conjunction with the {@link Retention} meta-annotation type to specify
* how long annotations are to be retained.
*
* @author  Joshua Bloch
* @since 1.5
*/
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

## @Repeatable

JDK 8 引入了 `@Repeatable`，表示该注解是否可重复标注。配套引入的还有为 `AnnotatedElement` 接口新增了两个方法 `getAnnotationsByType`、`getDeclaredAnnotationsByType`。

# AnnotatedElement

读取运行时（`@Retention(RetentionPolicy.RUNTIME)`）的注解，需要用到反射 API。`java.lang.reflect.AnnotatedElement` 接口提供了一组方法，用于获取注解信息：

![AnnotatedElement_methods](/img/java/reflection/AnnotatedElement_methods.png)

```java
// 判断某个注解是否存在
boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)
// 获取注解（包括父类）
<T extends Annotation> T getAnnotation(Class<T>)
<T extends Annotation> T[] getAnnotationsByType(Class<T>)
Annotation[] getAnnotations()
// 获取注解（不包括父类）
<T extends Annotation> T getDeclaredAnnotation(Class<T>)
<T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T>)
Annotation[] getDeclaredAnnotations()
```

由于下列类都实现了该接口，因此都拥有这些方法获取注解信息：

```java
java.lang.Class
java.lang.Package
java.lang.reflect.Field
java.lang.reflect.Method
java.lang.reflect.Constructor
java.lang.reflect.Parameter
```

![main_api](/img/java/reflection/main_api.png)

举个例子，看下哪些情况通过反射可以拿到注解（或拿不到）：

注解如下：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface Check {}
```

例子一：

```java
@Check
public class Person {
    @Check
    public void say() {}

    @Check
    public void say(String msg) {}
}

@ToString
public class Child extends Person {
    public String childName = "Hello";
    
    @Override
    public void say() {}
}

public class Test {
    @Test
    public void test() {
        ToString annotation = Child.class.getAnnotation(ToString.class);
        assertNull("结果为 null 因为 @ToString 标注为 @Retention(RetentionPolicy.SOURCE)", annotation);

        Check annotation1 = Child.class.getAnnotation(Check.class);
        assertNotNull("结果不为 null 因为 @Check 标注为 @Retention(RetentionPolicy.RUNTIME)、@Inherited 并且 @Check 被标注在父类上，可以被子类继承", annotation1);

        Check annotation2 = Child.class.getDeclaredAnnotation(Check.class);
        assertNull("结果为 null 因为 getDeclaredAnnotation 方法无法拿到父类注解", annotation2);

        Method sayString = Child.class.getMethod("say", String.class);
        Check annotation3 = sayString.getAnnotation(Check.class);
        assertNotNull("结果不为 null 因为 say 方法未被子类重写，被完整继承下来", annotation3);

        Method say = Child.class.getMethod("say");
        Check annotation4 = say.getAnnotation(Check.class);
        assertNull("结果为 null 因为 say 方法被子类重写了", annotation4);
    }
}
```

例子二：

```java
@Check
public interface Sayable {
    @Check
    void say();
}

public class Student implements Sayable {
    @Override
    public void say() {}
}

public class Test {
    @Test
    public void test() {
        Check annotation5 = Student.class.getAnnotation(Check.class);
        assertNull("结果为 null 因为接口上的注解无法被子类继承", annotation5);

        Check annotation6 = Student.class.getMethod("say").getAnnotation(Check.class);
        assertNull("结果为 null", annotation6);
    }
}
```

# AnnotatedType

JDK 8 引入了 `AnnotatedType`：

![AnnotatedType](/img/java/reflection/AnnotatedType.png)

类、字段、方法、构造方法相应增加了一组方法用于获取 `AnnotatedType`：

* `Class`

  ```java
  AnnotatedType getAnnotatedSuperclass()
  AnnotatedType[] getAnnotatedInterfaces()
  ```

* `Field`

  ```java
  AnnotatedType getAnnotatedType()
  ```

* `Method`

  ```java
  AnnotatedType getAnnotatedReturnType()
  AnnotatedType[] getAnnotatedParameterTypes()
  AnnotatedType[] getAnnotatedExceptionTypes()
  AnnotatedType getAnnotatedReceiverType()
  ```

* `Constructor`

  ```java
  AnnotatedType getAnnotatedReturnType()
  AnnotatedType[] getAnnotatedParameterTypes()
  AnnotatedType[] getAnnotatedExceptionTypes()
  AnnotatedType getAnnotatedReceiverType()
  ```

  

# 参考

《[深入理解java注解的实现原理](https://mp.weixin.qq.com/s?__biz=MzAxMjY1NTIxNA==&mid=2454441897&idx=1&sn=729688d470c94560c1e73e79f0c13adc&chksm=8c11e0a8bb6669be1cc4daee95b221ba437d536d598520d635fac4f18612dded58d6fddb0dce&scene=21#wechat_redirect)》

API：

- https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Annotation.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AnnotatedElement.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AnnotatedType.html