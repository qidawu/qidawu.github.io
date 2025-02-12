---
title: Java 集合框架系列（七）Map 接口的使用场景总结
date: 2018-05-13 22:29:30
updated:
tags: [Java, 数据结构]
typora-root-url: ..
---

`java.util.Map` 用于映射键值对（map keys to values）。

本文总结了几种 `Map` 的使用场景，例如：

* 如何实现有序遍历；
* 如何实现缓存淘汰策略；
* 如何解决 Top K 问题。
* ...

# 通用实现类

首先看看在 Java 中，默认为 `Map` 接口提供了哪些通用实现类：

## HashMap（散列表实现）

参考：《[Map 接口的散列表实现总结](/posts/java-collections-hashmap/)》

## TreeMap（红黑树实现）

红黑树实现。

## LinkedHashMap（散列表 + 双向链表实现）

`LinkedHashMap` 继承自 `HashMap`，是一种复合型数据结构。它在散列表的基础上，通过维护一个**有界队列（双向链表实现）**来实现「键值对」排序。

> 注意：`LinkedHashMap` 中的“Linked”实际上是指「链式队列」，而非指用链表法解决散列冲突。

这种行为适用于一些特定应用场景，例如：构建一个空间占用敏感的有限资源池，按某种淘汰策略自动淘汰「过期」元素：

| 排序方式                            | 使用场景               |
| ----------------------------------- | ---------------------- |
| 按插入顺序（`accessOrder = false`） | 实现 FIFO 缓存淘汰策略 |
| 按访问顺序（`accessOrder = true`）  | 实现 LRU 缓存淘汰策略  |

通过源码分析，`LinkedHashMap` 继承自 `HashMap`，同时 `LinkedHashMap` 的节点 `Entry` 也继承自 `HashMap` 的 `Node`，并且在此基础上增加了两个属性：

* 前驱节点 `Entry<K, V> before`
* 后继节点 `Entry<K, V> after`

![LinkedHashMap Entry](/img/java/collection/map/LinkedHashMap_Entry.png)

通过这两个属性就可以维护一条有序排列的双向链表，如下图：

![LinkedHashMap Entry](/img/java/collection/map/LinkedHashMap_Entry_sorted.png)

# 创建 Map

使用 Guava 快速创建不可变的 Map：

```java
Map<String, String> map = ImmutableMap.of("A", "Apple", "B", "Boy", "C", "Cat");
```

使用 Guava 快速创建指定 `initialCapacity` 初始容量的 Map：

```java
Maps.newHashMapWithExpectedSize(10);
```

Guava 的 `Maps` 还提供了更多的 API，可以自行研究使用。

# 实现有序遍历

## 集合视图

`Map` 接口提供了三种集合视图（*collection views*）：

![Map Views](/img/java/collection/map/Map_views.png)

> 注意：从 `keySet()` 返回类型可知，由于受限于 `Set` 的三大特性之一「互异性」，`Map` 不能包含重复的键（Key）。两个键重复与否取决于 [`equals`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-) 与[`hashCode()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--) 方法。

> Many methods in Collections Framework interfaces are defined in terms of the [`equals`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-) method. For example, the specification for the [`containsKey(Object key)`](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#containsKey-java.lang.Object-) method says: "returns `true` if and only if this map contains a mapping for a key `k` such that `(key==null ? k==null : key.equals(k))`." This specification should *not* be construed to imply that invoking `Map.containsKey` with a non-null argument `key` will cause `key.equals(k)` to be invoked for any key `k`. Implementations are free to implement optimizations whereby the `equals` invocation is avoided, for example, by first comparing the hash codes of the two keys. (The [`Object.hashCode()`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--) specification guarantees that two objects with unequal hash codes cannot be equal.) More generally, implementations of the various Collections Framework interfaces are free to take advantage of the specified behavior of underlying [`Object`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html) methods wherever the implementor deems it appropriate.

`hashCode` 需要遵循以下规则：

![equals_and_hashcode](/img/java/collection/map/alibaba_equals_and_hashcode.png)

## 遍历 API

Java 提供了下面几种 API 用于遍历 `Map` 接口的三种集合视图（*collection views*）：

> The *order* of a map is defined as the order in which the iterators on the map's *collection views* return their elements. Some map implementations, like the `TreeMap` class, make specific guarantees as to their order; others, like the `HashMap` class, do not.

### 内部循环 API

```java
// key 遍历
map.forEach((key, value) -> {});
```

### 外部循环 API

```java
// key 遍历
for (String key : map.keySet()) {}

// value 遍历
for (String value : map.values()) {}

// entry 遍历
for (Map.Entry<String, String> entry : map.entrySet()) {}
```

```java
// entry 显式迭代器
Iterator<Map.Entry<String, String>> it = map.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry<String, String> entry = it.next();
}

// foreach 循环增强，不再需要显式迭代器，简化迭代集合操作
for (Map.Entry<String, String> entry : map.entrySet()) {}
```

![map_entryset](/img/java/collection/map/alibaba_map_entryset.png)

## 遍历顺序

遍历顺序取决于 `Map` 接口使用了何种**数据结构**实现。

众所周知，散列表这种动态数据结构虽然支持非常高效的数据插入、删除、查找操作（平均时间复杂度都为常数阶 `O(1)`），但由于散列表中的数据都是**通过散列函数运算之后无序存储的**，因此散列表的遍历结果也是无序的，例如 `HashMap`。

要实现某种有序遍历的解决方案：

* 排序算法。每当我们希望按照某种顺序遍历散列表中的数据时，自行将散列表中的数据拷贝到数组中，然后通过某种排序算法完成排序，再进行遍历。但如此一来，效率势必会很低。

* 换一种数据结构实现，例如：红黑树（`TreeMap`）。

* 设计一种复合型数据结构，例如将散列表与链式队列结合在一起（`LinkedHashMap`），实现某种排序，从而达到有序遍历。

### 无序（no order）

```java
    /**
     * HashMap 按散列函数（取余运算）后获得的索引顺序进行存储
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

### 按自然顺序（natual order）

```java
    /**
     * TreeMap 按自然顺序进行排序（红黑树实现，查找的时间复杂度为 O(log(n))）
     * 
     * (4, 5)
     * (15, 3)
     * (16, 2)
     * (32, 1)
     * (65, 4)
     */
    @Test
    public void testTreeMap() {
        // Map<Integer, String> map = new TreeMap<>((o1, o2) -> o1.compareTo(o2));
        // Map<Integer, String> map = new TreeMap<>(Comparator.naturalOrder());
        Map<Integer, String> map = new TreeMap<>();
        map.put(32, "1");
        map.put(16, "2");
        map.put(15, "3");
        map.put(65, "4");
        map.put(4, "5");
        map.forEach((key, value) -> log.info("({}, {})", key, value));
    }
```

排序由键（Key）的自然顺序决定，通过 `Comparator` 或 `Comparable`。

### 按插入顺序（insertion order）

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

### 按访问顺序（access order）

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

# 实现缓存淘汰策略

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

# 实现合并

| Modifier and Type | Method and Description                                       |
| :---------------- | :----------------------------------------------------------- |
| `default V`       | `merge(K key, V value, BiFunction<? super V,? super V,? extends V> remappingFunction)`<br>If the specified key is not already associated with a value or is associated with null, associates it with the given non-null value. |

[Merging Two Maps with Java 8](https://www.baeldung.com/java-merge-maps)

# 实现计数器

Java 8 为 `Map` 接口引入了一组新的 `default` 默认方法，用于简化 `Map` 的日常使用。API 如下：

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| `default V`       | `putIfAbsent(K key, V value)`<br>If the specified key is not already associated with a value (or is mapped to `null`) associates it with the given value and returns `null`, else returns the current value. |
| `default V`       | `computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction)`<br>If the specified key is not already associated with a value (or is mapped to `null`), attempts to compute its value using the given mapping function and enters it into this map unless `null`. |
| `default V`       | `computeIfPresent(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)`<br>If the value for the specified key is present and non-null, attempts to compute a new mapping given the key and its current mapped value. |
| `default V`       | `compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)`<br>Attempts to compute a mapping for the specified key and its current mapped value (or `null` if there is no current mapping). |

要实现类似 Redis 散列表的原子递增命令 `HINCRBY` key field increment 的效果，使用 `compute` 实现的代码，对比传统代码更紧凑：

```Java
private static final Map<String, Integer> IP_STATS = new HashMap<>();

// 老版本
public synchronized int oldIpStats(String ip) {
    if (!IP_STATS.containsKey(ip)) {
        IP_STATS.put(ip, 1);
    } else {
        IP_STATS.put(ip, IP_STATS.get(ip) + 1);
    }
    return IP_STATS.get(ip);
}

// 新版本
public synchronized int newIpStats(String ip) {
    return IP_STATS.compute(ip, (key, oldValue) -> {
        if (oldValue == null) {
            return 1;
        } else {
            return oldValue + 1;
        }
    });
}
```

最终结果：

```Java
// result is 1
log.info("result is {}", newIpStats("127.0.0.1"));
// result is 2
log.info("result is {}", newIpStats("127.0.0.1"));
// result is 3
log.info("result is {}", newIpStats("127.0.0.1"));
```

# 解决 Top K 问题

解题思路：

* 数据规模大的，就先分而治之（hash 映射）；
* 数据规模小的：
  * 首先，按「键值对」保存统计数据（key 为关键字，value 为统计次数）
  * 然后，按「值」倒序
  * 最后，取 Top K 个「键」


```java
    /**
     * 统计 N 个随机数中最热门的 K 个
     */
    @Test
    public void topK() {
        int N = 100;
        // 「键值对」底层实现使用散列表即可，因为无须有序
        Map<Integer, Integer> numStats = Maps.newHashMapWithExpectedSize(N);

        Random random = new Random();
        for (int i = 0; i < N; i++) {
            int num = random.nextInt(10);
            // 首先，按「键值对」保存统计数据（key 为关键字，value 为统计次数）
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
        // 然后，按「值」倒序
        List<Map.Entry<Integer, Integer>> entries = new ArrayList<>(numStats.entrySet());
        entries.sort(Map.Entry.comparingByValue(Comparator.reverseOrder()));
        // 最后，取 Top K 个「键」
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

# 解码 Huffman coding

参考维基百科 [Huffman coding](https://en.wikipedia.org/wiki/Huffman_coding) 的定义：

> In [computer science](https://en.wikipedia.org/wiki/Computer_science) and [information theory](https://en.wikipedia.org/wiki/Information_theory), a **Huffman code** is a particular type of optimal [prefix code](https://en.wikipedia.org/wiki/Prefix_code) that is commonly used for [lossless data compression](https://en.wikipedia.org/wiki/Lossless_data_compression).

前缀编码（[prefix code](https://en.wikipedia.org/wiki/Prefix_code)） 的定义：

> 如果在一个编码方案中，任何一个编码都不是其它任何编码的前缀（最左子串），则称该编码是前缀编码。前缀编码可以保证对压缩数据进行解码时不产生二义性，确保正确解码。
>
> 前缀编码有两种编码方案：
>
> * 等长编码方案（[fixed-length coding](https://en.wikipedia.org/wiki/Prefix_code#Techniques)）
> * 不等长编码方案（[variable-length coding](https://en.wikipedia.org/wiki/Variable-length_code)）
>
> Huffman coding 是一种最优前缀编码，属于不等长编码方案。

参考：https://www.csdn.net/tags/MtTaMgysMjE3NjItYmxvZwO0O0OO0O0O.html



题目：已知字符对应的前缀编码表，给定一串编码，将其解码为字符串。

解答：https://blog.csdn.net/qq_45273552/article/details/109176832

# 参考

https://docs.oracle.com/javase/8/docs/api/java/util/Map.html

《[Guava 源码分析之Cache的实现原理](http://ifeve.com/guava-source-cache/)》

