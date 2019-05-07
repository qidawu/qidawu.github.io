---
title: Java 8 函数式编程
date: 2019-05-07 15:43:19
updated:
tags: Java
---

# 方法引用

| 方法引用类型 | 范例                     | Lambda 等式                                                |
| ------------ | ------------------------ | ---------------------------------------------------------- |
| 静态         | `Integer::parseInt`      | `str -> Integer.parseInt(str)`                             |
| 有限制       | `Instant.now()::isAfter` | `Instant then = Instant.now();`<br/>`t -> then.isAfter(t)` |
| 无限制       | `String::toLowerCase`    | `str -> str.toLowerCase()`                                 |
| 类构造器     | `TreeMap<K, V>::new`     | `() -> new TreeMap<K, V>`                                  |
| 数组构造器   | `int[]::new`             | `len -> new int[len]`                                      |

# 函数接口

| 接口                | 函数签名              | 范例                  | 范例               |
| ------------------- | --------------------- | --------------------- | ------------------ |
| `Predicate<T>`      | `boolean test(T t)`   | `String::isEmpty`     | 符合某个条件吗？   |
| `Supplier<T>`       | `T get()`             | `Instant::now`        | 无参的工厂方法     |
| `Consumer<T>`       | `void accept(T t)`    | `System.out::println` | 输出一个值         |
| `Function<T, R>`    | `R apply(T t)`        | `Arrays::asList`      | 类型转换           |
| `UnaryOperator<T>`  | `T apply(T t)`        | `String::toUpperCase` | 格式转换           |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | `BigInteger::add`     | 求两个数的加减乘除 |

## @FunctionInterface

`@FunctionInterface` 注解会强制 javac 检查一个接口是否符合函数接口的标准。如果该注释添加给一个枚举类型、类或另一个注解，或者接口包含不止一个抽象方法，javac 就会报错。重构代码时，使用它能很容易发现问题。

# Optional

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

`Optional` 源码：

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

