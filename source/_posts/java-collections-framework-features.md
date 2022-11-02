---
title: Java 集合框架系列（二）集合特性总结
date: 2018-04-14 09:13:11
updated:
tags: [Java, 数据结构]
typora-root-url: ..
---

集合接口中的许多修改方法都被标记为*可选（optional）*。实现类允许按需实现，未实现的方法需抛出运行时异常 `UnsupportedOperationException`。每个实现类的文档必须指明支持哪些可选操作。集合框架引入下列术语来帮助阐述本规范：

# 定长/变长

长度保证不变（即使元素可以更改）的列表称为 ***fixed-size*** 定长列表。反之则称为 ***variable-size*** 变长列表。

开发中接触最多的定长集合是通过 `Arrays.asList()` 创建的，该方法是一个适配器接口，将数组适配为定长列表，返回的对象是一个 `Arrays` 内部类，源码如下：

```java
    /**
     * Returns a fixed-size list backed by the specified array.  (Changes to
     * the returned list "write through" to the array.)  This method acts
     * as bridge between array-based and collection-based APIs, in
     * combination with {@link Collection#toArray}.  The returned list is
     * serializable and implements {@link RandomAccess}.
     *
     * <p>This method also provides a convenient way to create a fixed-size
     * list initialized to contain several elements:
     * <pre>
     *     List<String> stooges = Arrays.asList("Larry", "Moe", "Curly");
     * </pre>
     *
     * @param <T> the class of the objects in the array
     * @param a the array by which the list will be backed
     * @return a list view of the specified array
     */
    @SafeVarargs
    @SuppressWarnings("varargs")
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }

    private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
    {
        private static final long serialVersionUID = -2764017481108945198L;
        private final E[] a;

        ArrayList(E[] array) {
            a = Objects.requireNonNull(array);
        }

        ......

    }
```

分析发现，涉及元素增删的操作（如 `add()、remove()、clear()`）该内部类并没有实现，而是使用了父类 `AbstractList` 的方法，默认抛出 `UnsupportedOperationException` 异常：

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {

    public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }
    
    public E remove(int index) {
        throw new UnsupportedOperationException();
    }

}
```

参考手册：

![arrays_aslist](/img/java/collection/arrays_aslist.png)

# 可改/不可改

不支持修改操作（例如 `add`、`remove` 和 `clear`）的集合称为 ***unmodifiable*** 不可修改集合。反之则称为 ***modifiable*** 可修改集合。`Collections` 工具类提供了一组静态工厂方法，用于包装并返回指定集合的不可修改视图（*unmodifiable view*），如果尝试修改，则会抛出` UnsupportedOperationException`：

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

从源码分析，该包装类覆盖了所有修改方法并抛出异常 `UnsupportedOperationException`，实现非常简单：

```java
static class UnmodifiableCollection<E> implements Collection<E>, Serializable {
    private static final long serialVersionUID = 1820017752578914078L;

    final Collection<? extends E> c;

    UnmodifiableCollection(Collection<? extends E> c) {
        if (c==null)
            throw new NullPointerException();
        this.c = c;
    }

    public boolean add(E e) {
        throw new UnsupportedOperationException();
    }
    public boolean remove(Object o) {
        throw new UnsupportedOperationException();
    }
    public boolean addAll(Collection<? extends E> coll) {
        throw new UnsupportedOperationException();
    }
    public boolean removeAll(Collection<?> coll) {
        throw new UnsupportedOperationException();
    }
    public boolean retainAll(Collection<?> coll) {
        throw new UnsupportedOperationException();
    }
    public void clear() {
        throw new UnsupportedOperationException();
    }

    ......

}
```

# 可变/不可变

在 *unmodifiable* 的基础上，加之保证 `Collection` 实现类的底层数据为 `final` 的集合称为 ***immutable*** 不可变集合。反之则称为 ***mutable*** 可变集合。

Java 9 为 `List`、`Set` 和 `Map` 接口提供了新的静态工厂方法，可以创建这些集合的不可变实例，如下：

```java
List<String> list = List.of("apple", "orange", "banana");
Set<String> set = Set.of("aggie", "alley", "steely");
Map<String, String> map = Map.of("A", "Apple", "B", "Boy", "C", "Cat");
```

而 Java 9 之前，要实现不可变集合只能通过第三方库，例如用 Guava 实现相同效果：

```java
List<String> list = ImmutableList.of("apple", "orange", "banana");
Set<String> set = ImmutableSet.of("aggie", "alley", "steely");
Map<String, String> map = ImmutableMap.of("A", "Apple", "B", "Boy", "C", "Cat");
```

Guava 提供的不可变集合 API 如下：

```
ImmutableAsList
ImmutableBiMap
ImmutableClassToInstanceMap
ImmutableCollection
ImmutableEntry
ImmutableEnumMap
ImmutableEnumSet
ImmutableList
ImmutableListMultimap
ImmutableMap
ImmutableMapEntry
ImmutableMapEntrySet
ImmutableMapKeySet
ImmutableMapValues
ImmutableMultimap
ImmutableMultiset
ImmutableRangeMap
ImmutableRangeSet
ImmutableSet
ImmutableSetMultimap
ImmutableSortedAsList
ImmutableSortedMap
ImmutableSortedMapFauxverideShim
ImmutableSortedMultiset
ImmutableSortedMultisetFauxverideShim
ImmutableSortedSet
ImmutableSortedSetFauxverideShim
ImmutableTable
```

![immutable_collections](/img/java/collection/immutable_collections.PNG)

使用如下：

```java
List<String> list = ImmutableList.of("apple", "orange", "banana");
Set<String> set = ImmutableSet.of("aggie", "alley", "steely");
Map<String, String> map = ImmutableMap.of("A", "Apple", "B", "Boy", "C", "Cat");

log.info("list={}, set={}, map={}", fruits, marbles, map);  // list=[apple, orange, banana], set=[aggie, alley, steely], map={A=Apple, B=Boy, C=Cat}
```

除此之外，Apache Commons Lang 也提供了两个好用的类 `Pair` 和 `Triple`，可用于存放指定个数的临时数据：

```java
Triple.of("left", "middle", "right")
Pair.of("left", "right")
```

![methods_of_Pair_and_Triple](/img/java/collection/methods_of_Pair_and_Triple.png)

# 线程同步/非同步

参考[线程同步包装器](/2018/05/25/java-collections-framework-concurrent-impl/)

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

某些集合实现限制了可以存储哪些元素。可能的限制包括：

* 元素不能为 `null`。
* 元素必须属于特定类型。
* 元素必须匹配某些断言。

尝试添加违反集合实现限制的元素将导致运行时异常，如 `ClassCastException`、`IllegalArgumentException` 或 `NullPointerException`。

## 能否为 null

参考手册：

![map_element_of_null](/img/java/collection/map/alibaba_map_element_of_null.png)

## 类型限制

泛型机制虽然为集合提供了编译期类型检查，但仍然可以在运行期绕过此机制（通过反射也能绕过编译期类型检查）：

```java
public void test4() {
    List<Integer> intList = Lists.newArrayList(1, 2, 3);
    add(intList);
    
    // 循环到第四个元素时，报错：java.lang.ClassCastException: java.lang.String cannot be cast to java.lang.Integer
    for (int item : intList) {
        log.info("result is {}", item);
    }
}

private void add(List list) {
    list.add("4");
}
```

集合框架提供了一组包装器实现：

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

这些包装器实现用于返回指定集合的动态类型安全视图（*dynamically type-safe view*），核心源码如下：

```java
static class CheckedCollection<E> implements Collection<E>, Serializable {
    private static final long serialVersionUID = 1578914078182001775L;

    final Collection<E> c;
    final Class<E> type;

    @SuppressWarnings("unchecked")
    E typeCheck(Object o) {
        if (o != null && !type.isInstance(o))
            throw new ClassCastException(badElementMsg(o));
        return (E) o;
    }

    private String badElementMsg(Object o) {
        return "Attempt to insert " + o.getClass() +
            " element into collection with element type " + type;
    }

    CheckedCollection(Collection<E> c, Class<E> type) {
        this.c = Objects.requireNonNull(c, "c");
        this.type = Objects.requireNonNull(type, "type");
    }

    public boolean add(E e) { 
        return c.add(typeCheck(e));
    }

    ......

}
```

以 `add` 方法为例，每次添加元素时，都会调用 `typeCheck` 私有方法进行类型检查，如果尝试添加错误类型的元素，则会抛出 `ClassCastException`，通过 fail fast 防止后续出错：

```java
public void test() {
    List<Integer> intList = Collections.checkedList(Lists.newArrayList(1, 2, 3), Integer.class);
    add(intList);
    for (int item : intList) {
        log.info("result is {}", item);
    }
}

private void add(List list) {
    // java.lang.ClassCastException: Attempt to insert class java.lang.String element into collection with element type class java.lang.Integer
    list.add("4");
}
```

# 参考

《[Guava学习笔记：Immutable(不可变)集合](https://www.cnblogs.com/peida/p/Guava_ImmutableCollections.html)》

《[Java 中的 Mutable 和 Immutable](https://blog.csdn.net/Seriousplus/article/details/79750581)》（[en_US](http://web.mit.edu/6.031/www/sp17/classes/09-immutability/)）

https://stackoverflow.com/questions/7713274/java-immutable-collections

- Unmodifiable collections are usually read-only views (wrappers) of other collections. You can't add, remove or clear them, but **the underlying collection can change**.
- Immutable collections can't be changed at all - they don't wrap another collection - they have their own `final` elements.