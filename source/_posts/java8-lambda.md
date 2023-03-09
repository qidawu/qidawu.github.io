---
title: 函数式编程系列（一）Java 8 Lambda 表达式总结
date: 2019-05-07 15:43:19
updated:
tags: [函数式编程, Java]
typora-root-url: ..
---

# Lambda 表达式

Lambda 表达式总结：

![lambda](/img/java/lambda/lambda.png)

Java 8 为函数式编程新增的重点 API：

![api](/img/java/lambda/api.png)

# 函数式接口

函数式接口是只有一个抽象方法的接口，作为 Lambda 表达式和方法引用的[目标类型](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html#target-typing)。

JDK 8 新增了 9 组共 43 个通用型函数式接口，位于 `java.util.function` 包下，用来支持 Java 的函数式编程。接口如此之多的原因有二：

* 为了支持不同的参数个数。如 `UnaryOperator<T>` 仅支持一个参数，而 `BinaryOperator<T>` 支持两个参数。这一点从接口命名及函数签名也能看出：

   - `Unary` 一元
   - `Binary` 二元
   - `Ternary` 三元
   - `Quaternary` 四元
   - ......

* 泛型不支持原始数据类型。而在面对大数据量的流式 API 运算时，为了解决包装类在自动拆装箱的性能消耗，引入了 `int`、`long`、`Double` 原始数据类型的函数式接口。

   > 千万不要用带包装类型的基础函数接口来代替基本类型的函数接口。虽然可行，但它破坏了第 61 条的规则“基本类型优于装箱基本类型”。使用装箱基本类型进行批量操作处理，最终会导致致命的性能问题。——《Effective Java》

这些接口统计如下：

| 接口                  | 函数签名                 | 范例                  | 范例               | 基本类型特化                                                 |
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

方法引用通过方法的名字来指向一个方法，可以使语言的构造更紧凑简洁，进一步减少冗余代码，尤其是 Lambda 表达式。

方法引用使用一对冒号 `::`。

下面对比下方法引用简化 Lambda 表达式的例子：

| 方法引用类型 | 方法引用范例             | Lambda 表达式                                              |
| ------------ | ------------------------ | ---------------------------------------------------------- |
| 静态         | `Integer::parseInt`      | `str -> Integer.parseInt(str)`                             |
| 有限制       | `Instant.now()::isAfter` | `Instant then = Instant.now();`<br/>`t -> then.isAfter(t)` |
| 无限制       | `String::toLowerCase`    | `str -> str.toLowerCase()`                                 |
| 类构造器     | `TreeMap<K, V>::new`     | `() -> new TreeMap<K, V>`                                  |
| 数组构造器   | `int[]::new`             | `len -> new int[len]`                                      |

# 例子

这里对比 JavaScript 和 Java 两门语言的例子，方便对比学习。

## JavaScript 箭头函数

在 JavaScript 语言中，**函数是一等公民**（参考[为何钟爱一等公民]([https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/ch2.html#%E4%B8%BA%E4%BD%95%E9%92%9F%E7%88%B1%E4%B8%80%E7%AD%89%E5%85%AC%E6%B0%91](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/ch2.html#为何钟爱一等公民))、[知乎](https://www.zhihu.com/question/67652709)）。ES6 新特性允许使用“箭头”（`=>`）定义函数，语法简洁，使用如下：

没有参数，需要空括号：

```JavaScript
var f = () => 5;

// 等同于
var f = function () { return 5 };

// 5
var result = f();
```

一个参数，无需括号：

```JavaScript
var f = v => v;

// 等同于
var f = function (v) { return v; };

// 10
var result = f(10);
```

多个参数，需要括号：

```JavaScript
var sum = (num1, num2) => num1 + num2;

// 等同于
var sum = function(num1, num2) { return num1 + num2; };

// 20
var result = sum(10, 10);
```



箭头函数的一个用处是**简化回调函数**：

例子 1：

```JavaScript
// 正常函数写法
[1, 2, 3].forEach(function (x) {
  console.log(x);
})

// 箭头函数写法，结果 1 2 3
[1, 2, 3].forEach(x => console.log(x));
```

例子 2：

```JavaScript
// 正常函数写法
var result = [1, 2, 3].map(function (x) {
  return x * x;
});

// 箭头函数写法，结果 [1, 4, 9]
var result = [1, 2, 3].map(x => x * x);
```

例子 3：

```JavaScript
// 正常函数写法
var result = [2, 3, 1].sort(function (a, b) {
  return a - b;
});

// 箭头函数写法，结果 [1, 2, 3]
var result = [2, 3, 1].sort((a, b) => a - b);
```

例子 4：

```JavaScript
// 正常函数写法
var result = [1, 2, 3].filter(function (x) {
  return x > 1;
});

// 箭头函数写法，结果 [2, 3]
var result = [1, 2, 3].filter(x => x > 1);
```

## Java Lambda 表达式

然而在 Java 语言中，函数并非一等公民。但可以利用 Lambda 表达式 + 函数式接口来模拟 JavaScript 类似的语法，对比如下：

没有参数，需要空括号：

```Java
IntSupplier f = () -> 5;

// 5
int result = f.getAsInt();
```

一个参数，无需括号：

```Java
ToIntFunction<Integer> f = i -> i;

// 10
int result = f.applyAsInt(10);
```

多个参数，需要括号：

```Java
IntBinaryOperator sum = (num1, num2) -> num1 + num2;

// 使用方法引用进一步简化语法
// IntBinaryOperator sum = Integer::sum;

// 20
int result = sum.applyAsInt(10, 10);
```



Lambda 表达式同样可以**简化回调函数**：

例子 1：

```Java
// 1 2 3
IntStream.of(1, 2, 3).forEach(x -> System.out.println(x));

// 使用方法引用进一步简化语法
// IntStream.of(1, 2, 3).forEach(System.out::println);
```

例子 2：

```Java
// [1, 4, 9]
int[] result = IntStream.of(1, 2, 3).map(x -> x * x).toArray();
```

例子 3：

```Java
// [1, 2, 3]
int[] result = Stream.of(2, 3, 1)
    .sorted((a, b) -> a - b)
    .mapToInt(Integer::intValue)
    .toArray();

// 使用方法引用进一步简化语法
// int[] result = Stream.of(2, 3, 1)
//     .sorted(Comparator.naturalOrder())
//     .mapToInt(Integer::intValue)
//     .toArray();
```

例子 4：

```Java
// [2, 3]
int[] result = IntStream.of(1, 2, 3).filter(x -> x > 1).toArray();
```

# 使用场景

* Java 8 引入了 `Optional` 类用于解决臭名昭著的空指针异常。它本质上是一个可以为 `null` 的容器对象，并提供了很多有用的方法，以函数式编程的风格简化 `null` 处理。
* Stream API 是一种基于函数式编程的模型，用于增强集合处理。

# 参考

https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html

https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html

https://docs.oracle.com/javase/tutorial/collections/streams/index.html

《Effective Java 第三版》：

* 第 42 条：Lambda 优先于匿名类
* 第 43 条：方法引用优先于 Lambda
* 第 44 条：坚持使用标准的函数式接口（包括基本数据类型的函数式接口）
* 第 45 条：谨慎使用 Stream（必要时也需要使用 `Iterator` 外部迭代器）
* 第 46 条：优先选择 Stream 中无副作用的函数（使用收集器 `Collectors` 而不是 `forEach`）
* 第 47 条：Stream 要优先用 Collection 作为返回类型
* 第 48 条：谨慎使用 Stream 并行

《Java 8 函数式编程》

《Java 8 实战》

《[万字长文详解 Java lambda 表达式 | 阿里技术](https://mp.weixin.qq.com/s/ud5TckFLXWrVpilmobynhQ)》

《[Java8 Lambda 实现源码解析 | 阿里技术](https://mp.weixin.qq.com/s/d8BQ1-lEbeLuhZbn-t68yQ)》

