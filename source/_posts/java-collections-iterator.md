---
title: Java 集合框架系列（四）Iterator API 总结
date: 2018-05-01 23:46:35
updated:
tags: [Java, 数据结构]
typora-root-url: ..
---

本文总结下集合元素迭代的常用 API。

# 迭代器模式

> 迭代器模式（Iterator）是一种行为型设计模式，让你能在不暴露集合底层表现形式（列表、栈和树等）的情况下遍历集合中所有的元素。

![Iterator](/img/java/design-pattern/Iterator.png)

# 迭代器实现

在 Java 中，迭代器模式的实现有以下几种：

![Iterator_impl](/img/java/design-pattern/Iterator_impl.png)

* [`java.util.Enumeration<E>`](https://docs.oracle.com/javase/8/docs/api/java/util/Enumeration.html)：Java 1.0 引入，用于枚举集合元素。这种传统接口已被 `Iterator` 迭代器取代，虽然 `Enumeration` 还未被废弃，但在现代代码中已经被很少使用了。主要用于诸如 `java.util.Vector` 和 `java.util.Properties` 这些传统集合类。
* [`java.util.Iterator<E>`](https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html)：Java 1.2 引入。作为 [Java 集合框架](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html)的成员，迭代器取代了枚举。迭代器与枚举有两个不同之处：
  * 引入 `remove` 方法，允许调用者在迭代期间从集合中删除元素。
  * 方法名改进。
* [`java.lang.Iterable<T>`](https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html)：Java 1.5 引入。[For-each Loop](https://docs.oracle.com/javase/8/docs/technotes/guides/language/foreach.html) 语法糖的底层实现，实现这个接口的对象可以用于 "For-each Loop"语句，简化迭代器繁琐的使用语法。

上述三种迭代器实现都属于**命令式编程范式**，即使访问值的方法仅由迭代器负责实现。但实际上，是由开发者来决定何时访问序列中的 `next()` 项。

# 与 Stream API 对比

[`java.util.stream.Stream<T>`]()：Java 8 引入，用于实现 Stream API。与迭代器的区别在于：

* `Iterator` 外部迭代，使用命令式编程范式，完全由用户来决定”做什么“和”怎么做“，例如：

    ```java
    @Test
    public void iterator() {
        List<String> example = Arrays.asList("A", "B", "C");
        List<String> result = new ArrayList<>(example.size());
        // 怎么做（通过 Iterator API 遍历 List）
        Iterator<String> iterator = example.iterator();
        while (iterator.hasNext()) {
            // 做什么（把大写转成小写）
            result.add(iterator.next().toLowerCase());
        }
    }
    
    @Test
    public void iterable() {
        List<String> example = Arrays.asList("A", "B", "C");
        List<String> result = new ArrayList<>(example.size());
        // 怎么做（通过 For-each Loop 遍历 List）
        for (String s : example) {
            // 做什么（把大写转成小写）
            result.add(s.toLowerCase());
        }
    }
    ```

    ![outer_iterator](/img/java/collection/outer_iterator.png)

* `Stream` 内部迭代，使用声明式编程范式 > 函数式编程，用户仅需要决定“做什么”，而把“怎么做”的任务交给 JVM：

  ```java
  @Test
  public void streamApi() {
      List<String> result = Arrays.asList("A", "B", "C")
          // 这里遍历方式由 Stream API 实现，用户仅调用相应 API，数据量小可以用串行，数据量大可以用并行，怎么做、怎么实现用户并不用关心
          .stream()
          // 做什么（把大写转成小写）
          .map(String::toLowerCase)
          // 这里转成线性表由 Stream API 实现，用户仅调用相应 API，也可以转成集合、散列表，怎么实现用户并不关心。
          .collect(Collectors.toList());
  }
  ```
  
  ![inner_iterator](/img/java/collection/inner_iterator.png)

使用内部迭代的优势在于：

* 用户只需要关注问题本身，无需关注如何解决问题的细节。
* Java 可以利用短路、并行等对性能进行优化，用户无需关心。

# 与 Reactive Stream 对比

响应式编程范式通常在面向对象语言中作为**观察者模式**的扩展出现。可以将其与大家熟知的**迭代器模式**作对比，主要区别在于：

* 迭代器、Stream API 基于**拉模式（PULL）**
* 响应式流基于**推模式（PUSH）**

参考：[响应式编程总结](/2020/08/01/java-reactive-programming/)

# 参考

https://refactoringguru.cn/design-patterns/iterator