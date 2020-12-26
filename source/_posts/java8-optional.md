---
title: 函数式编程系列（二）Java 8 Optional API 总结
date: 2019-05-14 21:48:49
updated:
tags: [函数式编程, Java]
typora-root-url: ..
---

Java 8 引入了 `Optional` 类用于解决臭名昭著的空指针异常。它本质上是一个可以为 `null` 的容器对象，并提供了很多有用的方法，以函数式编程的风格简化 `null` 处理。

`Optional` 示例：

```java
String nullStr = null;

// 初始化 Optional
Optional<String> a = Optional.of("a");
Optional<String> b = Optional.empty();
Optional<String> c = Optional.ofNullable(nullStr);

String s0 = a.get();  // a
String s1 = b.orElse("other");  // other
String s2 = b.orElseGet(this::someExpensiveOperation);  // 方法引用的返回值
String s3 = b.orElseGet(() -> someExpensiveOperation());  // lambda 表达式的返回值
String s4 = b.orElseThrow(IllegalArgumentException::new);  // 抛异常
String s5 = b.orElseThrow(() -> new IllegalArgumentException("非法参数"));  // 抛异常

boolean isPresent = a.isPresent();  // true
a.ifPresent(System.out::println);  // a
a.filter(String::isEmpty).ifPresent(System.out::println);  // 条件不匹配，无打印
a.map(String::toUpperCase).ifPresent(System.out::println);  // 映射为大写字母 A 并打印
```

上述示例中，`Optional` 几个关键方法主要使用到这几个函数式接口：

* `orElseGet` 使用到： `java.util.function.Supplier`
* `orElseThrow` 使用到： `java.util.function.Supplier`
* `ifPresent` 使用到：`java.util.function.Consumer`
* `filter` 使用到：`java.util.function.Predicate`
* `map` 使用到：`java.util.function.Function`

`Optional` 几个关键方法的源码如下：

```java
public final class Optional<T> {

    /**
     * Return the value if present, otherwise invoke {@code other} and return
     * the result of that invocation.
     */
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }

    /**
     * Return the contained value, if present, otherwise throw an exception
     * to be created by the provided supplier.
     */
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }

    /**
     * If a value is present, invoke the specified consumer with the value,
     * otherwise do nothing.
     */
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }

    /**
     * If a value is present, and the value matches the given predicate,
     * return an {@code Optional} describing the value, otherwise return an
     * empty {@code Optional}.
     *
     */
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }

    /**
     * If a value is present, apply the provided mapping function to it,
     * and if the result is non-null, return an {@code Optional} describing the
     * result.  Otherwise return an empty {@code Optional}.
     */
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
}
```