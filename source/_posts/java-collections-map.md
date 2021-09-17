---
title: Java 集合框架系列（八）散列表常用场景总结
date: 2018-05-20 22:29:30
updated:
tags: Java
typora-root-url: ..
---

本文总结了几种散列表的常用场景，例如：

* 如何有序遍历散列表；
* 如何实现缓存淘汰策略；
* 如何解决海量数据的 Top K 问题。
* ...

# 遍历 API

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

# 排序方式

众所周知，散列表这种动态数据结构虽然支持非常高效的数据插入、删除、查找操作（时间复杂度都为常数阶 `O(1)`），但由于散列表中的数据都是**通过散列函数打乱之后无序存储的**，因此散列表遍历结果是无序的，例如 `HashMap`。

有两种有序遍历的解决方案：

* 实现排序算法。每当我们希望按照某种顺序遍历散列表中的数据时，自行将散列表中的数据拷贝到数组中，然后通过某种排序算法完成排序，再进行遍历。但如此一来，效率势必会很低。

* 设计一种复合型数据结构，将散列表与红黑树（例如 `TreeMap`）、与链表（例如 `LinkedHashMap`）、或者与跳表结合在一起，实现某种排序，从而达到有序遍历。

## 无序

```java
    /**
     * HashMap 按散列函数（取余）后的顺序进行排序
     *
     * (32, 1), 32 % 16 = 0
     * (16, 2), 16 % 16 = 0
     * (65, 4), 65 % 16 = 1
     * (4, 5),  4 % 16 = 4
     * (15, 3), 15 % 16 = 15
     */
    @Test
    public void testHashMap() {
        Map<Integer, String> map = new HashMap<>(16);
        map.put(32, "1");
        map.put(16, "2");
        map.put(15, "3");
        map.put(65, "4");
        map.put(4, "5");
        map.forEach((key, value) -> log.info("({}, {}), {} % 16 = {}", key, value, key, key % 16));
    }
```

## 按自然顺序

```java
    /**
     * TreeMap 按自然顺序进行排序
     * 
     * (4, 5)
     * (15, 3)
     * (16, 2)
     * (32, 1)
     * (65, 4)
     */
    @Test
    public void testTreeMap() {
        TreeMap<Integer, String> map = new TreeMap<>();
        map.put(32, "1");
        map.put(16, "2");
        map.put(15, "3");
        map.put(65, "4");
        map.put(4, "5");
        map.forEach((key, value) -> log.info("({}, {})", key, value));
    }
```

## 按插入顺序

```java
    /**
     * LinkedHashMap 大部分构造方法 默认按插入顺序进行排序（accessOrder=false）。底层维护了一个链式队列，可用于实现 FIFO 缓存淘汰策略
     *
     * (32, 1)
     * (16, 2)
     * (15, 3)
     * (65, 4)
     * (4, 5)
     */
    @Test
    public void testLinkedHashMap() {
        Map<Integer, String> map = new LinkedHashMap<>(16);
        map.put(32, "1");
        map.put(16, "2");
        map.put(15, "3");
        map.put(65, "4");
        map.put(4, "5");
        map.forEach((key, value) -> log.info("({}, {})", key, value));
    }
```

## 按访问顺序

```java
    /**
     * LinkedHashMap 唯一一个构造方法 用于按访问顺序进行排序（accessOrder=true）。底层维护了一个链式队列，可用于实现 LRU 缓存淘汰策略
     * (15, 3)
     * (65, 4)
     * (4, 5)
     * (16, 2)
     * (32, 1)
     */
    @Test
    public void testLinkedHashMap2() {
        Map<Integer, String> map = new LinkedHashMap<>(16, .75F, true);
        map.put(32, "1");
        map.put(16, "2");
        map.put(15, "3");
        map.put(65, "4");
        map.put(4, "5");

        map.get(16);
        map.get(32);

        map.forEach((key, value) -> log.info("({}, {})", key, value));
    }
```

# 缓存淘汰策略实现

## FIFO (First In First Out)

![FIFO (First In First Out)](/img/cache/FIFO.png)

```java
    /**
     * (1: one) removed
     * (2: two) removed
     * (3, three) hit
     * (4, four) hit
     * (5, five) hit
     */
    @Test
    public void FIFO() {
        Map<Integer, String> map = new LinkedHashMap<Integer, String>(3, .75F, false) {
            @Override
            protected boolean removeEldestEntry(Map.Entry eldest) {
                if (size() > 3) {
                    log.info("({}: {}) removed", eldest.getKey(), eldest.getValue());
                    return true;
                } else {
                    return false;
                }
            }
        };
        map.put(1, "one");
        map.put(2, "two");
        map.put(3, "three");
        map.get(1);
        map.put(4, "four");
        map.get(3);
        map.put(5, "five");
        map.forEach((key, value) -> log.info("({}, {}) hit", key, value));
    }
```

## LRU (Least Recently Used)

![LRU (Least Recently Used)](/img/cache/LRU.png)

```java
    /**
     * (2: two) removed
     * (1: one) removed
     * (4, four) hit
     * (3, three) hit
     * (5, five) hit
     */
    @Test
    public void LRU() {
        Map<Integer, String> map = new LinkedHashMap<Integer, String>(3, .75F, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry eldest) {
                if (size() > 3) {
                    log.info("({}: {}) removed", eldest.getKey(), eldest.getValue());
                    return true;
                } else {
                    return false;
                }
            }
        };
        map.put(1, "one");
        map.put(2, "two");
        map.put(3, "three");
        map.get(1);
        map.put(4, "four");
        map.get(3);
        map.put(5, "five");
        map.forEach((key, value) -> log.info("({}, {}) hit", key, value));
    }
```

## LFU (Least Frequently Used)

![LFU (Least Frequently Used)](/img/cache/LFU.png)

# 解决 Top K 问题

解题思路：

* 数据规模大的，就先分而治之（hash 映射）；
* 数据规模小的，直接 hash 统计 (关键字, 统计次数) + 排序（按统计次数倒序）。

```java
    /**
     * 统计 N 个随机数中最热门的 K 个
     */
    @Test
    public void topK() {
        int N = 100;
        Map<Integer, Integer> numStats = Maps.newHashMapWithExpectedSize(N);

        Random random = new Random();
        for (int i = 0; i < N; i++) {
            int num = random.nextInt(10);
            // 先 hash 统计 (关键字, 统计次数)
            numStats.compute(num, (key, oldVal) -> {
                if (oldVal == null) {
                    return 1;
                } else {
                    return oldVal + 1;
                }
            });
        }
        log.info("Before sort:");
        numStats.forEach((key, val) -> log.info("({}, {})", key, val));

        log.info("After sorted by statistics desc:");
        // 然后排序（按统计次数倒序）
        List<Map.Entry<Integer, Integer>> entries = new ArrayList<>(numStats.entrySet());
        entries.sort(Map.Entry.comparingByValue(Comparator.reverseOrder()));
        // 最后保留 Top K
        entries.forEach(entry -> log.info("({}, {})", entry.getKey(), entry.getValue()));
    }
```

输出结果：

```
Before sort:
(0, 12)
(1, 15)
(2, 6)
(3, 8)
(4, 12)
(5, 16)
(6, 5)
(7, 10)
(8, 9)
(9, 7)

After sorted by statistics desc:
(5, 16)
(1, 15)
(0, 12)
(4, 12)
(7, 10)
(8, 9)
(3, 8)
(9, 7)
(2, 6)
(6, 5)
```

# 数据结构实现

## TreeMap

红黑树。

## LinkedHashMap

`LinkedHashMap` 继承自 `HashMap`，是一种复合型数据结构。它在 `HashMap` 的基础上，通过维护一个**队列（双向链表）**来进行元素排序，从而实现以下两种排序方式：

| 排序方式   | 使用场景                 |
| ---------- | ------------------------ |
| 按插入顺序 | 可实现 FIFO 缓存淘汰策略 |
| 按访问顺序 | 可实现 LRU 缓存淘汰策略  |

通过源码分析，`LinkedHashMap` 继承自 `HashMap`，同时 `LinkedHashMap` 的节点 `Entry` 也继承自 `HashMap` 的 `Node`，并且在此基础上增加了两个属性：

* 前驱节点 `Entry<K, V> before`
* 后继节点 `Entry<K, V> after`

![LinkedHashMap Entry](/img/java/collection/LinkedHashMap_Entry.png)

通过这两个属性就可以维护一条有序排列的双向链表，如下图：

![LinkedHashMap Entry](/img/java/collection/LinkedHashMap_Entry_sorted.png)

# 参考

https://docs.oracle.com/javase/8/docs/api/java/util/Map.html

《[Guava 源码分析之Cache的实现原理](http://ifeve.com/guava-source-cache/)》
