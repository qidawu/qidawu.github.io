---
title: Java 8 函数式编程
date: 2019-05-07 15:43:19
updated:
tags: Java
typora-root-url: ..
---

# Lambda

![lambda](/img/java/lambda.png)

# 函数式接口

函数式接口是只有一个抽象方法的接口，用作 Lambda 表达式的类型。因此函数式接口可以被隐式转换为 lambda 表达式。

JDK 8 新增了 9 组共 43 个函数式接口，位于 `java.util.function` 包下，用来支持 Java 的函数式编程。接口如此之多的原因有二：

* 为了支持不同的参数个数。如 `UnaryOperator<T>` 仅支持一个参数，而 `BinaryOperator<T>` 支持两个参数。这一点从接口命名及函数签名也能看出：

   - `Unary` 一元
   - `Binary` 二元
   - `Ternary` 三元
   - `Quaternary` 四元

* 泛型不支持原始数据类型。而在面对大数据量的流式 API 运算时，为了解决包装类在自动拆装箱的性能消耗，引入了 `int`、`long`、`Double` 原始数据类型的函数式接口。

   > 千万不要用带包装类型的基础函数接口来代替基本类型的函数接口。虽然可行，但它破坏了第 61 条的规则“基本类型优于装箱基本类型”。使用装箱基本类型进行批量操作处理，最终会导致致命的性能问题。——《Effective Java》

这些接口统计如下：

| 接口                  | 函数签名                 | 范例                  | 范例               | 原始类型特化                                                 |
| --------------------- | ------------------------ | --------------------- | ------------------ | ------------------------------------------------------------ |
| `Predicate<T>`        | `boolean test(T t)`      | `String::isEmpty`     | 符合某个条件吗？   | `IntPredicate`<br/>`LongPredicate`<br/>`DoublePredicate`     |
| `BiPredicate<T, U>`   | `boolean test(T t, U u)` |                       |                    |                                                              |
| `Supplier<T>`         | `T get()`                | `Instant::now`        | 无参的工厂方法     | `BooleanSupplier`<br/>`IntSupplier`<br/>`LongSupplier`<br/>`DoubleSupplier` |
| `Consumer<T>`         | `void accept(T t)`       | `System.out::println` | 输出一个值         | `IntConsumer`<br/>`LongConsumer`<br/>`DoubleConsumer`        |
| `BiConsumer<T, U>`    | `void accept(T t, U u)`  |                       |                    | `ObjIntConsumer<T>`<br/>`ObjLongConsumer<T>`<br/>`ObjDoubleConsumer<T>` |
| `Function<T, R>`      | `R apply(T t)`           | `Arrays::asList`      | 类型转换           | `IntFunction<R>`<br/>`IntToLongFunction`<br/>`IntToDoubleFunction`<br/>`LongFunction<R>`<br/>`LongToIntFunction`<br/>`LongToDoubleFunction`<br/>`DoubleFunction<R>`<br/>`DoubleToIntFunction`<br/>`DoubleToLongFunction`<br/>`ToIntFunction<T>`<br/>`ToLongFunction<T>`<br/>`ToDoubleFunction<T>` |
| `BiFunction<T, U, R>` | `R apply(T t, U u)`      |                       |                    | `ToIntBiFunction<T, U>`<br/>`ToLongBiFunction<T, U>`<br/>`ToDoubleBiFunction<T, U>` |
| `UnaryOperator<T>`    | `T apply(T t)`           | `String::toUpperCase` | 格式转换           | `IntUnaryOperator`<br/>`LongUnaryOperator`<br/>`DoubleUnaryOperator` |
| `BinaryOperator<T>`   | `T apply(T t1, T t2)`    | `BigInteger::add`     | 求两个数的加减乘除 | `IntBinaryOperator`<br/>`LongBinaryOperator`<br/>`DoubleBinaryOperator` |

以上接口都标注了 `@FunctionalInterface`。这是 Java 8 为函数式接口引入的一个新注解，有两个目的：

* 告诉这个接口及其文档的读者，这个接口是针对 Lambda 设计的；
* 用于编译级错误检查。加上该注解，当你写的接口不符合函数式接口定义的时候，编译器会报错。该注解会强制 `javac` 检查一个接口是否符合函数式接口的标准。如果该注释添加给一个枚举类型、类或另一个注解，**或者接口包含不止一个抽象方法**，`javac` 就会报错。重构代码时，使用它能很容易发现问题，因此建议必须始终用 `@FunctionalInterface` 注解对自己编写的函数式接口进行标注。

此外，函数式接口允许：

* 函数式接口里允许定义默认方法，因为默认方法不是抽象方法，其有一个默认实现，所以是符合函数式接口的定义的。
  
* 函数式接口里允许定义静态方法，因为静态方法不能是抽象方法，是一个已经实现了的方法，所以是符合函数式接口的定义的。
  
* 函数式接口里允许定义 `java.lang.Object` 里的 `public` 方法。

# 方法引用

特点：

- 通过方法的名字来指向一个方法
- 方法引用使用一对冒号 `::`
- 可以使语言的构造更紧凑简洁，减少冗余代码

| 方法引用类型 | 范例                     | Lambda 等式                                                |
| ------------ | ------------------------ | ---------------------------------------------------------- |
| 静态         | `Integer::parseInt`      | `str -> Integer.parseInt(str)`                             |
| 有限制       | `Instant.now()::isAfter` | `Instant then = Instant.now();`<br/>`t -> then.isAfter(t)` |
| 无限制       | `String::toLowerCase`    | `str -> str.toLowerCase()`                                 |
| 类构造器     | `TreeMap<K, V>::new`     | `() -> new TreeMap<K, V>`                                  |
| 数组构造器   | `int[]::new`             | `len -> new int[len]`                                      |

# 使用场景

## Optional

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

## Stream API

Stream 并不只是一个 API，它是一种基于函数编程的模型。

# 总结

引述自《Effective Java》：

* 第 42 条：Lambda 优先于匿名类
* 第 43 条：方法引用优先于 Lambda
* 第 44 条：坚持使用标准的函数式接口（包括基本数据类型的函数式接口）
* 第 45 条：谨慎使用 Stream（必要时也需要使用 `Iterator` 外部迭代器）
* 第 46 条：优先选择 Stream 中无副作用的函数（使用收集器 `Collectors` 而不是 `forEach`）
* 第 47 条：Stream 要优先用 Collection 作为返回类型
* 第 48 条：谨慎使用 Stream 并行