---
title: Java 集合框架系列（一）框架总结
date: 2018-04-10 22:19:15
updated:
tags: [Java, 数据结构]
typora-root-url: ..
---

集合（*collection*）表示一组对象。Java SE 提供了集合框架（*collections framework*），是一个用于表示和操作集合的统一框架，使集合可以独立于实现细节进行操作。集合框架的主要优点如下：

- 通过提供数据结构和算法实现，使用户无需自行编写，**减少编程工作**。
- 通过提供数据结构和算法的高性能实现来**提高性能**。由于每个接口的各种实现是可互换的，因此可以通过切换实现来调整和优化程序。
- 通过提供一套标准接口来**促进软件重用**。
- 通过建立一种通用语言，**为无关联的集合 API 之间提供互操作性**。
- **减少学习成本**，只须学习一些特设的集合 API。

集合框架的整体组成如下：

![overview](/img/java/collection/overview.png)

下面分别来看下各组成部分。

# 集合接口

集合接口分为下面两组，这些接口构成了集合框架的基础：

* `java.util.Collection`，表示一组对象集合

  > A collection represents a group of objects, known as its *elements*. 
  >
  > * Some collections allow duplicate elements and others do not. 
  >
  > * Some are ordered and others unordered. 
  >
  > The JDK does not provide any *direct* implementations of this interface: it provides implementations of more specific subinterfaces like `Set` and `List`. 
  > 
  > This interface is typically used to pass collections around and manipulate them where maximum generality is desired.

* `java.util.Map`，用于存储键值对

  > An object that maps keys to values.
  >
  > * A map cannot contain duplicate keys; 
  >
  > * each key can map to at most one value.

## Collection

[Collection](https://en.wikipedia.org/wiki/Collection_(abstract_data_type))

最基础的集合接口 `java.util.Collection` 及其子接口如下：

![Collection](/img/java/collection/Collection.png)

其中，常用的五个重点接口的方法及使用要点如下：

![methods_of_collection](/img/java/collection/methods_of_collection.png)

## Map

[Map](https://en.wikipedia.org/wiki/Associative_array)

其它集合接口基于 `java.util.Map`，不是真正的集合。但是，这些接口包含集合视图（*collection-view*）操作，使得它们可以作为集合进行操作。

![Map](/img/java/collection/map/Map.png)

`java.util.Map` 接口的方法如下:

![Map methods](/img/java/collection/map/Map_methods.png)

# 集合实现类

## 抽象类实现

下列抽象类为核心集合接口提供了基本功能实现，以最小化用户自定义实现的成本。这些抽象类的 API 文档精确地描述了各个方法的实现方式，实现者能够参阅并了解哪些方法需要覆盖：

![Collection_abstract_class](/img/java/collection/Collection_abstract_class.png)

## 通用实现

`java.util.Collection` 的通用实现如下：

![collection_impl](/img/java/collection/collection_impl.png)

`java.util.Map` 的通用实现如下：

![map_impl](/img/java/collection/map/Map_impl.jpg)

集合接口的主要实现，命名通常形如 <*Implementation-style*><*Interface*>。通用实现类汇总如下（左列为接口，表头为数据结构）：

|         | **Resizable Array**                                          | **Linked List**                                              | **Hash Table**                                               | **Hash Table + Linked List**                                 | **Balanced Tree**                                            | **Heap**                                                     |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `List`  | [`ArrayList`](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html) | [`LinkedList`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html) |                                                              |                                                              |                                                              |                                                              |
| `Queue` | `ArrayBlockingQueue`                                         | [`LinkedList`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html)<br/>`LinkedBlockingQueue`<br/>`LinkedTransferQueue`<br/>`ConcurrentLinkedQueue` |                                                              |                                                              |                                                              | `PriorityBlockingQueue`<br/>[`PriorityQueue`](https://docs.oracle.com/javase/8/docs/api/java/util/PriorityQueue.html) |
| `Deque` | [`ArrayDeque`](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayDeque.html) | [`LinkedList`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html)<br/>`LinkedBlockingDeque`<br/>`ConcurrentLinkedDeque` |                                                              |                                                              |                                                              |                                                              |
| `Set`   |                                                              |                                                              | [`HashSet`](https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html) | [`LinkedHashSet`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashSet.html) | [`TreeSet`](https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html) |                                                              |
| `Map`   |                                                              |                                                              | [`HashMap`](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html) | [`LinkedHashMap`](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html) | [`TreeMap`](https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html) |                                                              |

通用实现的特性如下：

* 通用实现类支持集合接口中的所有可选操作，并且对包含的元素没有限制。
* 都是非线程同步的。`Collections` 工具类提供了称为[*同步包装器（synchronization wrappers）*](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedCollection-java.util.Collection-)的静态工厂方法可用于添加同步行为。
* 所有集合实现都具有*快速失败的迭代器（fail-fast iterators）*，可以检测到无效的并发修改，然后快速失败，而不是表现异常。

## 遗留实现

早期版本的集合类，已被改进以实现新的集合接口：

- `java.util.Vector` - `List` 接口的可变长数组实现，线程同步，包含其它遗留方法。
- `java.util.Hashtable` -  `Map` 接口的散列表实现，线程同步，键和值都不允许为 `null`，包含其它遗留方法。继承自抽象类 `java.util.Dictionary`。

## 并发实现

为高并发使用而设计的实现。详见另一篇《并发实现总结》。

## 特殊实现

用于特殊情况的实现：

* `CopyOnWriteArrayList` 写时复制列表
* `CopyOnWriteArraySet` 写时复制列表
* `WeakHashMap`
* `IdentityHashMap`
* `EnumSet`
* `EnumMap`

## 适配器实现（Adaptor）

将某个集合接口适配成另一个：

- 根据 `Map` 的通用实现创建一个 `Set` 的通用实现：

  ```java
  Collections.newSetFromMap(Map)
  ```

- 以后进先出（Lifo）队列的形式返回 `Deque` 的视图：

  ```java
  Collections.asLifoQueue(Deque)
  ```

* 将数组转换为 `List` 集合：

  ```java
  Arrays.asList(...)
  ```

## 包装器实现（Wrapper）

用于其它集合实现的功能增强：

* 返回指定集合的不可修改视图（*unmodifiable view*），如果尝试修改，则会抛出` UnsupportedOperationException`：

  ```java
  Collections.unmodifiableCollection
  Collections.unmodifiableSet
  Collections.unmodifiableSortedSet
  Collections.unmodifiableNavigableSet
  Collections.unmodifiableList
  Collections.unmodifiableMap
  Collections.unmodifiableSortedMap
  Collections.unmodifiableNavigableMap
  ```

* 返回由指定集合支持的 `synchronized` 线程同步集合：

  ```java
  Collections.synchronizedCollection
  Collections.synchronizedSet
  Collections.synchronizedSortedSet
  Collections.synchronizedNavigableSet
  Collections.synchronizedList
  Collections.synchronizedMap
  Collections.synchronizedSortedMap
  Collections.synchronizedNavigableMap
  ```

* 返回指定集合的动态类型安全视图（*dynamically type-safe view*），如果尝试添加错误类型的元素，则会抛出 ` ClassCastException`。泛型机制虽然提供了编译期类型检查，但可以绕过此机制。动态类型安全试图消除了这种可能性：

  ```java
  Collections.checkedCollection
  Collections.checkedQueue
  Collections.checkedSet
  Collections.checkedSortedSet
  Collections.checkedNavigableSet
  Collections.checkedList
  Collections.checkedMap
  Collections.checkedSortedMap
  Collections.checkedNavigableMap
  ```

## 便利实现

集合接口的高性能版“迷你实现”：

* 返回一个不可变集合（*immutable*），不包含任何元素：

  ```java
  Collections.emptySet
  Collections.emptySortedSet
  Collections.emptyNavigableSet
  Collections.emptyList
  Collections.emptyMap
  Collections.emptySortedMap
  Collections.emptyNavigableMap
  ```

* 返回一个不可变集合（*immutable*），仅包含一个元素：

  ```java
  Collections.singleton
  Collections.singletonList
  Collections.singletonMap
  ```

* 返回一个不可变集合（*immutable*），包含指定元素的 N 个拷贝：

  ```java
  Collections.nCopies
  ```

* 返回一个由指定数组支持的定长集合（*fixed-size*）：

  ```java
  Arrays.asList
  ```

# 基础设施

为集合接口提供必要支持的接口。例如：

* 迭代器 `Iterator`、`ListIterator`
* 排序接口 `Comparable`、`Comparator`
* 运行时异常 `UnsupportedOperationException`、`ConcurrentModificationException`
* 标记接口 `RandomAccess`

# 算法和工具实现

**算法实现**。由工具类 `Collections` 提供，用于集合，提供了很多静态方法例如 `sort` 排序、`binarySearch` 查找、`replaceAll` 替换等。这些算法体现了多态性，因为相同的方法可以在相似的接口上有着不同的实现。

**数组工具**。由工具类 `Arrays` 提供，用于基本类型和引用类型数组，提供了很多静态方法例如 `sort` 排序、`binarySearch` 查找等。严格来说，这些工具不是集合框架的一部分，此功能在集合框架引入的同时被添加到 Java 平台，并依赖于一些相同的基础设施。

# 参考

https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html

[Why Java Collection Framework doesn't contain Tree and Graph ?](https://stackoverflow.com/questions/4978487/why-java-collection-framework-doesnt-contain-tree-and-graph)

https://www.baeldung.com/category/java/

- https://www.baeldung.com/category/java/tag/java-array/
- https://www.baeldung.com/category/java/tag/java-set/
- https://www.baeldung.com/category/java/tag/java-list/
- https://www.baeldung.com/category/java/tag/java-map/