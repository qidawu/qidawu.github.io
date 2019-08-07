---
title: Java 集合框架系列（四）各版本功能增强总结
date: 2019-04-28 00:26:19
updated:
tags: Java
typora-root-url: ..
---

Java 集合框架并不是一蹴而就写成的，也是经过了好多个版本迭代的演进与发展，才走到今天。本文总结下集合框架各版本的功能增强。

# Java SE 9

`List`、`Set` 和 `Map` 接口中，新的静态工厂方法可以创建这些集合的*不可变实例（immutable）*，如下：

```java
List<String> list = List.of("apple", "orange", "banana");
Set<String> set = Set.of("aggie", "alley", "steely");
Map<String, String> map = Map.of("A", "Apple", "B", "Boy", "C", "Cat");
```

参考：https://www.linuxidc.com/Linux/2017-10/147683.htm

# Java SE 8

* [Lambda 表达式](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
* Stream API，参考《[Java 8 中处理集合的优雅姿势——Stream](https://mp.weixin.qq.com/s/adKZrOe6nFEmuADHijsAtA)》
* [聚合操作](https://docs.oracle.com/javase/tutorial/collections/streams/)，例如 `forEach`
* 作为 [JEP 180](http://openjdk.java.net/jeps/180) 提案的成果，`HashMap`、`LinkedHashMap`、`ConcurrentHashMap` 的性能得到提升。当出现大量散列冲突时，值将存储在红黑树而不是链表，以提升查找性能。

# Java SE 7

- 新增一个集合接口：`TransferQueue`，以及实现类 `LinkedTransferQueue`。
- 为 `Map` 及其派生实现类引入了一个性能改进的替代版散列函数（但在 Java SE 8 已被移除并取代）。

# Java SE 6

新增几个集合接口：

* `Deque`
* `BlockingDeque`
* `NavigableSet`
* `NavigableMap`
* `ConcurrentNavigableMap`

新增几个集合实现类：

* `ArrayDeque`
* `ConcurrentSkipListSet`
* `ConcurrentSkipListMap`
* `LinkedBlockingDeque`
* `AbstractMap.SimpleEntry`
* `AbstractMap.SimpleImmutableEntry`

现有实现类增强：

* `LinkedList` 实现 `Deque` 接口
* `TreeSet` 实现 `NavigableSet` 接口
* `TreeMap` 实现 `NavigableMap` 接口

`Collections` 工具类新增两个适配器方法：

* `newSetFromMap(Map)` 根据 `Map` 的通用实现创建一个 `Set` 的通用实现
* `asLifoQueue(Deque)` 以后进先出（Lifo）队列的形式返回 `Deque` 的视图。

`Arrays` 工具类新增两个方法：

* `copyOf`
* `copyOfRange`

# Java SE 5

三个新增的**语法糖**显著增强了集合框架：

* 泛型：为集合框架添加编译时类型安全，并在读取元素时不再需要做类型转换。
* 自动装箱/拆箱：往集合插入元素时自动装箱（将原始数据类型转换为对应的包装类型），读取元素时自动拆箱。
* 增强 `for` 循环：迭代集合时不再需要显式迭代器（`Iterator`）。

  ```java
  // 数组迭代
  String[] strArray = {"apple", "orange", "banana"};
  for (String s : strArray) {
      System.out.println(s);
  }
  
  // List 迭代
  List<String> strList = Arrays.asList("apple", "orange", "banana");
  for (String s : strList) {
      System.out.println(s);
  }
  ```

通用实现与并发实现：

* 新增三个集合接口：
  * `Queue`
  * `BlockingQueue`
  * `ConcurrentMap`
* 新增几个 `Queue` 实现类：
  * `PriorityQueue`
  * `ConcurrentLinkedQueue`
  * `LinkedList` 实现 `Queue` 接口
  * `AbstractQueue` 抽象类实现
* 新增五个 `BlockingQueue` 实现类，位于 `java.util.concurrent` 包下：
  * `LinkedBlockingQueue`
  * `ArrayBlockingQueue`
  * `PriorityBlockingQueue`
  * `DelayQueue`
  * `SynchronousQueue`
* 新增一个 `ConcurrentMap` 实现类：
  * `ConcurrentHashMap`

特殊实现：

* 新增两个特殊用途的 `List` 和 `Set` 实现类，用于读远大于写以及迭代无法线程同步的情况：
  * `CopyOnWriteArrayList`
  * `CopyOnWriteArraySet`
* 新增两个特殊用途的 `Set` 和 `Map` 实现类，用于枚举：
  * `EnumSet`
  * `EnumMap`

包装器实现：

* 新增一位包装器实现家族的新成员 `Collections.checkedInterface` ，主要用于通用集合。

`Collections` 工具类新增三个通用算法和一个 `Comparator` 转换器：

* `frequency(Collection<?> c, Object o)` 计算指定元素在指定集合中出现的次数。
* `disjoint(Collection<?> c1, Collection<?> c2)` 求两个集合是否不相交。
* `addAll(Collection<? super T> c, T... a)` 将指定数组中的所有元素添加到指定集合的便捷方法。
* `Comparator<T> reverseOrder(Comparator<T> cmp)` 反向排序。

`Arrays` 工具类新增下列方法：

* `hashCode`、`toString`
* `deepEquals`、`deepHashCode`、`deepToString` 用于多维数组

# Java SE 1.4

* `Collections` 工具类新增几个新方法，例如 ：
  * `replaceAll(List list, Object oldVal, Object newVal)` 查找替换。
* 新增标记接口 `RandomAccess`。
* 新增集合实现类 `LinkedHashMap`、`LinkedHashSet`。内部使用散列表 + 双向链表（按插入顺序排序）。

# 参考

http://openjdk.java.net/jeps/180