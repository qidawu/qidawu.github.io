---
title: Java 集合框架系列（二）术语规范总结
date: 2019-04-08 09:13:11
updated:
tags: Java
typora-root-url: ..
---

集合接口中的许多修改方法都被标记为*可选（optional）*。实现类允许按需实现，未实现的方法需抛出运行时异常 `UnsupportedOperationException`。每个实现类的文档必须指明支持哪些可选操作。集合框架引入下列术语来帮助阐述本规范：

# 可改/不可改

不支持修改操作（例如 `add`、`remove` 和 `clear`）的集合称为 ***unmodifiable*** 不可修改集合。反之则称为 ***modifiable*** 可修改集合。`Collections` 工具类提供了一组静态工厂方法，用于包装并返回指定集合的不可修改视图（*unmodifiable view*），如果尝试修改，则会抛出` UnsupportedOperationException`。

# 可变/不可变

在 *unmodifiable* 的基础上，加之保证 `Collection` 实现类的底层数据为 `final` 的集合称为 ***immutable*** 不可变集合。反之则称为 ***mutable*** 可变集合。

Java 9 之前，要实现不可变集合只能通过第三方库，例如 Guava：

```java
List<String> list = ImmutableList.of("apple", "orange", "banana");
Set<String> set = ImmutableSet.of("aggie", "alley", "steely");
Map<String, String> map = ImmutableMap.of("A", "Apple", "B", "Boy", "C", "Cat");

log.info("list={}, set={}, map={}", fruits, marbles, map);  // list=[apple, orange, banana], set=[aggie, alley, steely], map={A=Apple, B=Boy, C=Cat}
```

Java 9 为 `List`、`Set` 和 `Map` 接口提供了新的静态工厂方法，可以创建这些集合的不可变实例，如下：

```java
List<String> list = List.of("apple", "orange", "banana");
Set<String> set = Set.of("aggie", "alley", "steely");
Map<String, String> map = Map.of("A", "Apple", "B", "Boy", "C", "Cat");
```

参考：

* 《[Guava学习笔记：Immutable(不可变)集合](https://www.cnblogs.com/peida/p/Guava_ImmutableCollections.html)》
* 《[Java 中的 Mutable 和 Immutable](https://blog.csdn.net/Seriousplus/article/details/79750581)》（[en_US](http://web.mit.edu/6.031/www/sp17/classes/09-immutability/)）
* https://stackoverflow.com/questions/7713274/java-immutable-collections
  * Unmodifiable collections are usually read-only views (wrappers) of other collections. You can't add, remove or clear them, but **the underlying collection can change**.
  * Immutable collections can't be changed at all - they don't wrap another collection - they have their own `final` elements.

# 定长/变长

长度保证不变（即使元素可以更改）的列表称为 ***fixed-size*** 定长列表。反之则称为 ***variable-size*** 变长列表。

# 随机/顺序访问

支持根据下标索引快速（时间复杂度 `0(1)`）访问元素的列表称为 ***random access*** 随机访问列表。反之则称为 ***sequential access*** 顺序访问列表。

标记接口 `java.util.RandomAccess` 用于标记列表类支持随机访问，其实现类如下：

![RandomAccess](/img/java/collection/RandomAccess.png)

该标记接口使得 `Collections` 工具类中的通用算法实现能够据此更改其行为以提升性能：

* `binarySearch`
* `reverse`
* `shuffle`
* `fill`
* `copy`
* `rotate`
* `replaceAll`
* `indexOfSubList`
* `lastIndexOfSubList`
* `checkedList`

以 `binarySearch` 为例，源码判断如下：

```java
    public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```

# 元素限制

某些集合实现限制了可以存储哪些元素（例如 `Map` 的 key 和 value ）。可能的限制包括：

* 元素必须属于特定类型。
* 元素不能为 `null`。
* 元素必须匹配某些断言。

尝试添加违反集合实现限制的元素将导致运行时异常，如 `ClassCastException`、`IllegalArgumentException` 或 `NullPointerException`。