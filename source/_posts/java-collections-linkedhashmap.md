---
title: Java 集合框架系列（八）LinkedHashMap 有序散列表实现总结
date: 2018-05-20 22:29:30
updated:
tags: Java
typora-root-url: ..
---

# 遍历散列表

Java 提供了下面几种 API 用于遍历散列表：

## 内部循环 API

```java
map.forEach((key, value) -> {});
```

## 外部循环 API

```java
// key 迭代
for (String key : map.keySet()) {}

// value 迭代
for (String value : map.values()) {}

// entry 显式迭代器
Iterator<Map.Entry<String, String>> it = map.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry<String, String> entry = it.next();
}

// foreach 循环增强，不再需要显式迭代器，简化迭代集合操作
for (Map.Entry<String, String> entry : map.entrySet()) {}
```

![map_entryset](/img/java/collection/map_entryset.png)

# 如何顺序遍历散列表？

但是，众所周知，散列表这种动态数据结构虽然支持非常高效的数据插入、删除、查找操作（时间复杂度都为常数阶 `O(1)`），但由于散列表中的数据都是**通过散列函数打乱之后无序存储的**，因此无法按照某种顺序遍历数据。

有两种解决方案：

* 实现排序算法。每当我们希望按照某种顺序遍历散列表中的数据时，自行将散列表中的数据拷贝到数组中，然后通过某种排序算法完成排序，再进行遍历。但如此一来，效率势必会很低。

* 设计一种复合型数据结构，例如将散列表和链表（或者跳表）结合在一起，实现顺序遍历。

# LinkedHashMap

`LinkedHashMap` 继承自 `HashMap`，正是这样一种复合型数据结构。它在 `HashMap` 的基础上，通过维护一条**双向链表**来进行元素排序，从而实现以下两种顺序遍历方式：

## 按插入顺序遍历

```java
public static void main(String[] args) {
    Map<String, String> map = new LinkedHashMap<String, String>();
    map.put("apple", "苹果");
    map.put("watermelon", "西瓜");
    map.put("banana", "香蕉");
    map.put("peach", "桃子");

    Iterator iter = map.entrySet().iterator();
    while (iter.hasNext()) {
        Map.Entry entry = (Map.Entry) iter.next();
        System.out.println(entry.getKey() + "=" + entry.getValue());
    }
}
```

输出结果：

```
apple=苹果
watermelon=西瓜
banana=香蕉
peach=桃子
```

## 按访问顺序遍历

```java
public static void main(String[] args) {
    Map<String, String> map = new LinkedHashMap<String, String>(16,0.75f,true);
    map.put("apple", "苹果");
    map.put("watermelon", "西瓜");
    map.put("banana", "香蕉");
    map.put("peach", "桃子");

    // 被访问的元素置底
    map.get("banana");
    map.get("apple");

    Iterator iter = map.entrySet().iterator();
    while (iter.hasNext()) {
        Map.Entry entry = (Map.Entry) iter.next();
        System.out.println(entry.getKey() + "=" + entry.getValue());
    }
}
```

输出结果：

```
watermelon=西瓜
peach=桃子
banana=香蕉
apple=苹果
```

## 数据结构分析

通过源码分析，`LinkedHashMap` 继承自 `HashMap`，同时 `LinkedHashMap` 的节点 `Entry` 也继承自 `HashMap` 的 `Node`，并且在此基础上增加了两个属性：

* 前驱节点 `Entry<K, V> before`
* 后继节点 `Entry<K, V> after`

![LinkedHashMap Entry](/img/java/collection/LinkedHashMap_Entry.png)

通过这两个属性就可以维护一条有序排列的双向链表，如下图：

![LinkedHashMap Entry](/img/java/collection/LinkedHashMap_Entry_sorted.png)

## 源码分析

