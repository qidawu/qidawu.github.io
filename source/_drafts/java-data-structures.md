---
title: Java 数据结构总结
date: 2019-07-21 22:19:15
updated:
tags: Java
typora-root-url: ..
---

# 前言

# 集合接口

集合接口分为两组。最基础的 [`java.util.Collection`](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html) 接口有下列子接口：

- `java.util.Set`
  - `java.util.SortedSet`
  - `java.util.NavigableSet`
- `java.util.Queue`
  - `java.util.concurrent.BlockingQueue`
  - `java.util.concurrent.TransferQueue`
- `java.util.Deque`
  - `java.util.concurrent.BlockingDeque`



# 集合实现类

集合接口的实现类命名通常形如 <*Implementation-style*><*Interface*>。通用实现类汇总如下（左列为接口，表头为实现类）：

|         | **Resizable Array**                                          | **Linked List**                                              | **Hash Table**                                               | **Hash Table + Linked List**                                 | **Balanced Tree**                                            |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `List`  | [`ArrayList`](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html) | [`LinkedList`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html) |                                                              |                                                              |                                                              |
| `Deque` | [`ArrayDeque`](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayDeque.html) | [`LinkedList`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html) |                                                              |                                                              |                                                              |
| `Set`   |                                                              |                                                              | [`HashSet`](https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html) | [`LinkedHashSet`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashSet.html) | [`TreeSet`](https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html) |
| `Map`   |                                                              |                                                              | [`HashMap`](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html) | [`LinkedHashMap`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html) | [`TreeMap`](https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html) |

# 并发集合

# 设计目标

# 参考

https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html