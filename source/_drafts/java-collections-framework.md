---
title: Java 集合框架总结
date: 2019-04-06 22:19:15
updated:
tags: Java
typora-root-url: ..
---

# 前言

集合（*collection*）表示一组对象。Java SE 提供了集合框架（*collections framework*），是一个用于表示和操作集合的统一框架，使集合可以独立于实现细节进行操作。集合框架的主要优点如下：

- 通过提供数据结构和算法实现，使用户无需自行编写，**减少编程工作**。
- 通过提供数据结构和算法的高性能实现来**提高性能**。由于每个接口的各种实现是可互换的，因此可以通过切换实现来调整和优化程序。
- 通过提供一套标准接口来**促进软件重用**。
- 通过建立一种通用语言，**为无关联的集合 API 之间提供互操作性**。
- **减少学习成本**，只须学习一些特设的集合 API。

# 集合框架组成

整体组成如下：

![overview](/img/java/collection/overview.png)

## 接口

**集合接口**。表示不同类型的集合，例如 `Set`、`List` 和 `Map`。这些接口构成了框架的基础。

**基础设施**。为集合接口提供必要支持的接口。例如：
* 迭代器（`Iterator`、`ListIterator`）
* 排序接口（`Comparable`、`Comparator`）
* 运行时异常（`UnsupportedOperationException`、`ConcurrentModificationException`）
* 标记接口 `RandomAccess`

## 实现

**通用实现**。集合接口的主要实现。

**遗留实现**。早期版本的集合类，例如 `Vector` 和 `Hashtable` ，已被改进以实现新的集合接口。

* `Vector`
* `Hashtable`

**特殊实现**。用于特殊情况的实现。

**并发实现**。为高并发使用而设计的实现。

**包装器实现**。向集合实现添加各种功能：

* `Collections.unmodifiableInterface` 返回指定集合的不可修改视图（*unmodifiable view*），如果尝试修改，则会抛出` UnsupportedOperationException`。
* `Collections.synchronizedInterface` 返回由指定集合支持的线程同步集合。
* `Collections.checkedInterface` 返回指定集合的动态类型安全视图（*dynamically type-safe view*），如果客户端尝试添加错误类型的元素，则会抛出 ` ClassCastException`。泛型机制虽然提供了编译期类型检查，但可以绕过此机制。动态类型安全视图消除了这种可能性。

**适配器实现**。将某个集合接口适配成另一个：

- `Collections.newSetFromMap(Map)` 根据 `Map` 的通用实现创建一个 `Set` 的通用实现
- `Collections.asLifoQueue(Deque)` 以后进先出（Lifo）队列的形式返回 `Deque` 的视图。

**便利实现**。集合接口的高性能版“迷你实现”：

* `Arrays.asList`
* `emptySet, emptyList and emptyMap`
* `singleton, singletonList, and singletonMap`
* `nCopies`

**抽象类实现**。集合接口的部分功能实现，有助于自定义实现。

## 算法和工具实现

**算法实现**。由工具类 `Collections` 提供，用于集合，提供了很多静态方法例如 `sort` 排序、`binarySearch` 查找、`replaceAll` 替换等。这些算法体现了多态性，因为相同的方法可以在相似的接口上有着不同的实现。

**数组工具**。由工具类 `Arrays` 提供，用于基本类型和引用类型数组，提供了很多静态方法例如 `sort` 排序、`binarySearch` 查找等。严格来说，这些工具不是集合框架的一部分，此功能在集合框架引入的同时被添加到 Java 平台，并依赖于一些相同的基础设施。

# 集合接口

集合接口分为两组：`java.util.Collection` 和 `java.util.Map`：

## Collection

最基础的集合接口 `java.util.Collection` 及其子接口如下：

![Collection](/img/java/collection/Collection.png)

## Map

其它集合接口基于 `java.util.Map`，不是真正的集合。但是，这些接口包含集合视图（*collection-view*）操作，使得它们可以作为集合进行操作。

![Map](/img/java/collection/Map.png)

# 集合接口特性

集合接口中的许多修改方法都被标记为*可选（optional）*。实现类允许按需实现，未实现的方法需抛出运行时异常 `UnsupportedOperationException`。每个实现类的文档必须指明支持哪些可选操作。集合框架引入下列术语来帮助阐述本规范：

## 可改/不可改

不支持修改操作（例如 `add`、`remove` 和 `clear`）的集合称为 ***unmodifiable*** 不可修改集合。反之则称为 ***modifiable*** 可修改集合。

## 可变/不可变

在 *unmodifiable* 的基础上，加之保证 `Collection` 实现底层数据为 `final` 的集合称为 ***immutable*** 不可变集合。反之则称为 ***mutable*** 可变集合。

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

* 《[Java 中的 Mutable 和 Immutable](https://blog.csdn.net/Seriousplus/article/details/79750581)》（[en_US](http://web.mit.edu/6.031/www/sp17/classes/09-immutability/)）

* https://stackoverflow.com/questions/7713274/java-immutable-collections
  * Unmodifiable collections are usually read-only views (wrappers) of other collections. You can't add, remove or clear them, but **the underlying collection can change**.
  * Immutable collections can't be changed at all - they don't wrap another collection - they have their own `final` elements.
* 《[Guava学习笔记：Immutable(不可变)集合](https://www.cnblogs.com/peida/p/Guava_ImmutableCollections.html)》

## 定长/变长

长度保证不变（即使元素可以更改）的列表称为 ***fixed-size*** 定长列表。反之则称为 ***variable-size*** 变长列表。

## 随机访问/顺序方法

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

## 元素限制

某些集合实现限制了可以存储哪些元素（例如 `Map` 的 key 和 value ）。可能的限制包括：

* 元素必须属于特定类型。
* 元素不能为 `null`。
* 元素必须匹配某些断言。

尝试添加违反集合实现限制的元素将导致运行时异常，如 `ClassCastException`、`IllegalArgumentException` 或 `NullPointerException`。

# 集合实现类

## 抽象实现

下列抽象类为核心集合接口提供了基本功能实现，以最小化用户自定义实现的成本。这些抽象类的 API 文档精确地描述了各个方法的实现方式，因此实现者能够参阅并了解哪些方法需要覆盖：

![Collection_abstract_class](/img/java/collection/Collection_abstract_class.png)

## 通用实现

集合接口的实现类命名通常形如 <*Implementation-style*><*Interface*>。通用实现类汇总如下（表头为数据结构，左列为接口）：

|         | **Resizable Array**                                          | **Linked List**                                              | **Hash Table**                                               | **Hash Table + Linked List**                                 | **Balanced Tree**                                            | **Heap**                                                     |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `List`  | [`ArrayList`](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html) | [`LinkedList`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html) |                                                              |                                                              |                                                              |                                                              |
| `Queue` |                                                              |                                                              |                                                              |                                                              |                                                              | [`PriorityQueue`](https://docs.oracle.com/javase/8/docs/api/java/util/PriorityQueue.html) |
| `Deque` | [`ArrayDeque`](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayDeque.html) | [`LinkedList`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html) |                                                              |                                                              |                                                              |                                                              |
| `Set`   |                                                              |                                                              | [`HashSet`](https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html) | [`LinkedHashSet`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashSet.html) | [`TreeSet`](https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html) |                                                              |
| `Map`   |                                                              |                                                              | [`HashMap`](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html) | [`LinkedHashMap`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html) | [`TreeMap`](https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html) |                                                              |

通用实现类支持集合接口中的所有可选操作，并且对包含的元素没有限制。它们都是非线程同步的，但 `Collections` 工具类包含称为[*同步包装器（synchronization wrappers）*](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedCollection-java.util.Collection-)的静态工厂方法可用于向许多非线程同步的集合添加同步行为。

所有集合实现都具有*快速失败的迭代器（fail-fast iterators）*，可以检测到无效的并发修改，然后快速失败，而不是表现异常。

# 并发集合

并发编程时，多线程并发操作集合必须小心谨慎。Java SE 提供了各种并发友好的集合接口和实现类。这些接口和类超越了前面讨论的*同步包装器（synchronization wrappers）*，以提供并发编程中经常需要的功能。

并发集合接口如下：

- `BlockingQueue`
- `TransferQueue`
- `BlockingDeque`
- `ConcurrentMap`
- `ConcurrentNavigableMap`

并发集合实现类如下：

![concurrent_collections](/img/java/collection/concurrent_collections.png)

## 队列

有两种并发队列实现，阻塞式、非阻塞式：

![Queue_implementations](/img/java/collection/Queue_implementations.png)

### 阻塞队列

阻塞队列实现：

![BlockingQueue](/img/java/collection/BlockingQueue_implementations.png)

阻塞双端队列实现：

![BlockingDeque_implementations](/img/java/collection/BlockingDeque_implementations.png)

### 并发队列（非阻塞）

`ConcurrentLinkedQueue`

`ConcurrentLinkedDeque`

## 并发散列表

`ConcurrentHashMap`

![ConcurrentHashMap](/img/java/collection/ConcurrentHashMap.png)

## 并发跳表

`ConcurrentSkipListSet`

`ConcurrentSkipListMap`

## 写时复制列表

`CopyOnWriteArrayList`

`CopyOnWriteArraySet`

# 各版本功能增强

## Java SE 9

`List`、`Set` 和 `Map` 接口中，新的静态工厂方法可以创建这些集合的*不可变实例（immutable）*，如下：

```java
List<String> list = List.of("apple", "orange", "banana");
Set<String> set = Set.of("aggie", "alley", "steely");
Map<String, String> map = Map.of("A", "Apple", "B", "Boy", "C", "Cat");
```

参考：https://www.linuxidc.com/Linux/2017-10/147683.htm

## Java SE 8

* [Lambda 表达式](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
* Stream API，参考《[Java 8 中处理集合的优雅姿势——Stream](https://mp.weixin.qq.com/s/adKZrOe6nFEmuADHijsAtA)》
* [聚合操作](https://docs.oracle.com/javase/tutorial/collections/streams/)，例如 `forEach`
* 作为 [JEP 180](http://openjdk.java.net/jeps/180) 提案的成果，`HashMap`、`LinkedHashMap`、`ConcurrentHashMap` 的性能得到提升。当出现大量散列冲突时，值将存储在红黑树而不是链表，以提升查找性能。

## Java SE 7

- 新增一个集合接口：`TransferQueue`，以及实现类 `LinkedTransferQueue`。
- 为 `Map` 及其派生实现类引入了一个性能改进的替代版散列函数（但在 Java SE 8 已被移除并取代）。

## Java SE 6

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

## Java SE 5

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

## Java SE 1.4

* `Collections` 工具类新增几个新方法，例如 ：
  * `replaceAll(List list, Object oldVal, Object newVal)` 查找替换。
* 新增标记接口 `RandomAccess`。
* 新增集合实现类 `LinkedHashMap`、`LinkedHashSet`。内部使用散列表 + 双向链表（按插入顺序排序）。

# 参考

https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html

http://openjdk.java.net/jeps/180

https://blog.csdn.net/way2016/article/details/93380850

https://www.cnblogs.com/gujiande/p/9485493.html