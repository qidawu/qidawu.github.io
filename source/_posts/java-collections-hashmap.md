---
title: Java 集合框架系列（六）HashMap 散列表
date: 2019-08-16 22:29:30
updated:
tags: Java
typora-root-url: ..
---

散列表是一种通过散列函数可以快速定位数据的数据结构。散列表利用了数组支持按照下标随机访问数据的时候，时间复杂度为 O(1) 的特性，**所以散列表其实就是数组的一种扩展，由数组演化而来**。可以说，如果没有数组，就没有散列表。

![hashtable_examples](/img/java/collection/hashtable_examples.png)

# 内部类

## Entry 接口设计

`java.util.Map` 接口中的内部接口 `Entry` 如下，其中 `getKey`、`getValue`、`setValue`、`equals`、`hashCode` 五个方法待实现：

![Entry](/img/java/collection/Map_Entry.png)

## 单链表节点实现

![Node](/img/java/collection/HashMap_Node.png)

```java
    /**
     * Basic hash bin node, used for most entries.  (See below for
     * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
     */
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) && Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final String toString() { return key + "=" + value; }
    }
```

## 红黑树节点实现

![TreeNode](/img/java/collection/HashMap_TreeNode.png)

```java
    /**
     * Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
     * extends Node) so can be used as extension of either regular or
     * linked node.
     */
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;

        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }

        ......

    }
```

# 常量及字段

![HashMap_fields](/img/java/collection/HashMap_fields.png)

## 常量值

```java
    /**
     * The default initial capacity - MUST be a power of two.
     * 数组默认初始容量为 16，必须是 2 的 n 次幂
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     *
     * 数组最大容量，，必须是 2 的 n 次幂，且小于等于 2^30
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     * 默认装载因子
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     *
     * 桶树化时的阈值
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     *
     * 桶非树化时的阈值
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     * 
     * 树化时数组最小容量
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```

## 字段

关键的几个字段：

* `table` 散列表的底层数据结构为数组，其容量用 `capacity` 表示。此外，Java 通过链表法解决散列冲突的问题，因此数组类型为 `Node<K,V>[]` 节点数组。
* `loadFactor` 散列表的装载因子。当散列表中空闲位置不多的时候，散列冲突的概率就会大大提高。为了尽可能保证散列表的操作效率，一般情况下，会尽可能保证散列表中有一定比例的空闲槽位。我们用装载因子(load factor)来表示空位的多少。装载因子越大，表示空闲槽位越少，冲突越多，散列表的性能会下降。计算公式：`装载因子 = 键值对数量 size / 数组容量 capacity`。
* `threshold` 散列表的扩容阈值，计算公式：`threshold = capacity * load factor`，即默认配置下的扩容阈值为：`12=16*0.75`。
* `size` 散列表实际存储的键值对数量，如果 `size > threshold` 则进行双倍扩容并重新散列。
* `modCount` 修改次数

```java
    /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     *
     * 底层数组
     */
    transient Node<K,V>[] table;

    /**
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * The number of key-value mappings contained in this map.
     *
     * 实际存储的键值对数量
     */
    transient int size;

    /**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     *
     * 修改次数
     */
    transient int modCount;

    /**
     * The next size value at which to resize (capacity * load factor).
     *
     * resize 操作的阈值（capacity * load factor）
     */
    int threshold;

    /**
     * The load factor for the hash table.
     *
     * 装载因子
     */
    final float loadFactor;
```

# 构造方法

构造方法的参数主要用于指定数组的初始容量 `initialCapacity`、装载因子 `loadFactor`，并计算扩容阈值 `threshold`。

![HashMap_constructors](/img/java/collection/HashMap_constructors.png)

```java
    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        // 参数校验及默认值设置
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
        
        // 装载因子及扩容阈值赋值
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```

## 初始容量

在集合初始化时，建议指定 `initialCapacity` 初始容量：

![map_constructor](/img/java/collection/map_constructor.png)

如果不想手工计算初始容量 `initialCapacity`，可以使用 Guava 的静态工厂方法 `Maps.newHashMapWithExpectedSize`，源码如下：

```java
  /**
   * Creates a {@code HashMap} instance, with a high enough "initial capacity"
   * that it <i>should</i> hold {@code expectedSize} elements without growth.
   * This behavior cannot be broadly guaranteed, but it is observed to be true
   * for OpenJDK 1.7. It also can't be guaranteed that the method isn't
   * inadvertently <i>oversizing</i> the returned map.
   *
   * @param expectedSize the number of entries you expect to add to the
   *        returned map
   * @return a new, empty {@code HashMap} with enough capacity to hold {@code
   *         expectedSize} entries without resizing
   * @throws IllegalArgumentException if {@code expectedSize} is negative
   */
  public static <K, V> HashMap<K, V> newHashMapWithExpectedSize(int expectedSize) {
    return new HashMap<K, V>(capacity(expectedSize));
  }

  /**
   * Returns a capacity that is sufficient to keep the map from being resized as
   * long as it grows no larger than expectedSize and the load factor is >= its
   * default (0.75).
   */
  static int capacity(int expectedSize) {
    if (expectedSize < 3) {
      checkNonnegative(expectedSize, "expectedSize");
      return expectedSize + 1;
    }
    if (expectedSize < Ints.MAX_POWER_OF_TWO) {
      // This is the calculation used in JDK8 to resize when a putAll
      // happens; it seems to be the most conservative calculation we
      // can make.  0.75 is the default load factor.
      return (int) ((float) expectedSize / 0.75F + 1.0F);
    }
    return Integer.MAX_VALUE; // any large value
  }
```

不管是通过手工指定还是通过 Guava 的静态工厂方法，计算出来的初始容量都只是一个参考值。因为在随后 `resize` 会重新计算。

## 扩容阈值

第一个构造方法中调用了 `tableSizeFor` 方法，用于产生一个大于等于 `initialCapacity` 的最小的 2 的整数次幂。做法是通过右移操作让每一位都变为 1，最后 +1 变成 2 的 n 次幂：

```java
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

# 关键私有方法

## 散列函数

整个散列函数的整体过程如下：

```java
int hash(Object key) {
    int h = key.hashCode()；
    return (h ^ (h >>> 16)) & (capitity - 1); //capicity 表示散列表的大小
}
```

在 `HashMap` 源码中，是分两步走的：

第一步，hash 值的计算，源码如下：

```java
    /**
     * Computes key.hashCode() and spreads (XORs) higher bits of hash
     * to lower.  Because the table uses power-of-two masking, sets of
     * hashes that vary only in bits above the current mask will
     * always collide. (Among known examples are sets of Float keys
     * holding consecutive whole numbers in small tables.)  So we
     * apply a transform that spreads the impact of higher bits
     * downward. There is a tradeoff between speed, utility, and
     * quality of bit-spreading. Because many common sets of hashes
     * are already reasonably distributed (so don't benefit from
     * spreading), and because we use trees to handle large sets of
     * collisions in bins, we just XOR some shifted bits in the
     * cheapest possible way to reduce systematic lossage, as well as
     * to incorporate impact of the highest bits that would otherwise
     * never be used in index calculations because of table bounds.
     */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

该函数也叫扰动函数：先获取 32 位长度的 hashCode，再进行高 16 位与低 16 位异或，在低位加入高位特征，目的是减少碰撞的概率。

其中，`hashCode` 需要遵循以下规则：

![equals_and_hashcode](/img/java/collection/equals_and_hashcode.png)

第二步，在更新、插入或删除的时候，计算 Key 被映射到的桶的位置：

```java
// capacity 表示 table.length
int index = hash(key) & (capacity - 1)
```

该按位与运算相当于通过**取模法（除留余数法）**计算存放的数组下标。

总结：

> JDK HashMap中hash函数的设计，确实很巧妙：
>
> 首先hashcode本身是个32位整型值，在系统中，这个值对于不同的对象必须保证唯一（JAVA规范），这也是大家常说的，重写equals必须重写hashcode的重要原因。
>
> 获取对象的hashcode以后，先进行移位运算，然后再和自己做异或运算，即：hashcode ^ (hashcode >>> 16)，这一步甚是巧妙，是将高16位移到低16位，这样计算出来的整型值将“具有”高位和低位的性质。
>
> 最后，用hash表当前的容量减去一，再和刚刚计算出来的整型值做位与运算。进行位与运算，很好理解，是为了计算出数组中的位置。但这里有个问题：
>
> 为什么要用容量减去一？
>
> 因为 A % B = A & (B - 1)，注意这个公式只有当B为2的幂次方时才有效，所以HashMap中的数组容量要求必须是2的幂次方。
>
> 最后，(h ^ (h >>> 16)) & (capitity -1) = (h ^ (h >>> 16)) % capitity，可以看出这里本质上是使用了「除留余数法」
>
> 综上，可以看出，hashcode的随机性，加上移位异或算法，得到一个非常随机的hash值，再通过「除留余数法」，得到index

## 设值（putVal）

一些关键代码分析：

* 散列表只在首次设值时，才初始化；
* 散列函数：`(n - 1) & hash`，通过**取模法**计算存放的数组下标。通过该散列函数将元素的键值（key）映射为数组下标，然后将数据存储在数组中对应的下标位置。当我们按照键值查询元素时，用同样的散列函数，将键值转化为数组下标，从对应的数组下标的位置取数据。因此散列函数在散列表中起着非常关键的作用。
* 为了避免散列值冲突，除了对比散列值是否相等之外，还需要对比 `key` 是否相等；
* 散列冲突解决方法：链表法。同时为了避免链表过长及**散列表碰撞攻击**，如果节点数超过 8 个，则进行树化 `treeifyBin`，避免大量散列冲突导致散列表退化成单链表，导致查询时间复杂度从 0(1) 退化成 0(n)；
* 如果实际映射数量超过阈值，则进行 `resize` 扩容。

```java
    /**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        // tab 表示底层数组
        // n 表示数组长度
        // i 表示数组下标
        // p 表示头结点
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 节点数组为空，则首次初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;

        // 计算数组下标，并获取指定下标元素，如果为空，表示无散列冲突，则创建节点并放入指定下标对应的桶
        // 无散列冲突情况：
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        // 有散列冲突情况：
        else {
            // e 表示当前节点
            Node<K,V> e; K k;
            // 判断头结点是否为重复键，如果是则后续覆盖值
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 如果是根节点是树节点类型，则进行相关操作
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 否则就是单链表节点
            else {
                // 遍历单链表
                for (int binCount = 0; ; ++binCount) {
                    // 找到尾结点
                    if ((e = p.next) == null) {
                        // 创建新节点并追加到单链表末尾
                        p.next = newNode(hash, key, value, null);
                        // 如果节点数超过 8 个，则进行树化，避免大量散列冲突导致散列表退化成单链表，导致查询时间复杂度从 0(1) 退化成 0(n)
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            // 树化操作可能会引起数组容量翻倍、扩容阈值翻倍
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 如果桶中的节点有重复键，则后续覆盖值
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 如果存在重复键，则覆盖值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 实际数量加一。同时如果超过阈值，则进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

树化前，会先判断数组容量是否达到阈值 `MIN_TREEIFY_CAPACITY = 64`，如果是则将指定散列值对应数组下标的桶中所有链表节点都替换为红黑树节点，否则进行 `resize` 扩容操作：

```java
    /**
     * Replaces all linked nodes in bin at index for given hash unless
     * table is too small, in which case resizes instead.
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

## 查找节点（getNode）

关键的 `getNode` 查找节点方法：

```java
    /**
     * Implements Map.get and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // 判断散列表是否为空，且散列值对应头节点是否为空
        if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
            // 头节点散列值、键值匹配，则返回该节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            // 否则遍历下一个节点
            if ((e = first.next) != null) {
                // 树节点
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                // 顺序遍历单链表，查找匹配结果
                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

## 删除节点（removeNode）

关键的 `removeNode` 删除节点方法：

```java
    /**
     * Implements Map.remove and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to match if matchValue, else ignored
     * @param matchValue if true only remove if value is equal
     * @param movable if false do not move other nodes while removing
     * @return the node, or null if none
     */
    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;

                ++modCount;
                --size;
                afterNodeRemoval(node);

                return node;
            }
        }
        return null;
    }
```

# 常用方法

## 设值

```java
    @Override
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    @Override
    public V putIfAbsent(K key, V value) {
        return putVal(hash(key), key, value, true, true);
    }

    /**
     * Copies all of the mappings from the specified map to this map.
     * These mappings will replace any mappings that this map had for
     * any of the keys currently in the specified map.
     *
     * @param m mappings to be stored in this map
     * @throws NullPointerException if the specified map is null
     */
    @Override
    public void putAll(Map<? extends K, ? extends V> m) {
        putMapEntries(m, true);
    }

    /**
     * Implements Map.putAll and Map constructor
     *
     * @param m the map
     * @param evict false when initially constructing this map, else
     * true (relayed to method afterNodeInsertion).
     */
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            if (table == null) { // pre-size
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            else if (s > threshold)
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

## 查找和替换

下面这组方法都使用到了关键的 `getNode` 方法查找指定节点：

```java
    @Override
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }


    @Override
    public V getOrDefault(Object key, V defaultValue) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
    }

    @Override
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }

    @Override
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }

    @Override
    public boolean replace(K key, V oldValue, V newValue) {
        Node<K,V> e; V v;
        if ((e = getNode(hash(key), key)) != null &&
            ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
            e.value = newValue;
            afterNodeAccess(e);
            return true;
        }
        return false;
    }

    @Override
    public V replace(K key, V value) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) != null) {
            V oldValue = e.value;
            e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
        return null;
    }
```

## 删除节点

```java
    @Override
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }

    @Override
    public boolean remove(Object key, Object value) {
        return removeNode(hash(key), key, value, true, true) != null;
    }
```

## 清空 map

```java
    @Override
    public void clear() {
        Node<K,V>[] tab;
        modCount++;
        
        // 内部数组不为空，则遍历并清空所有元素
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
```

## 迭代

外部循环：

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

内部循环：

```java
map.forEach((key, value) -> {});
```

![map_entryset](/img/java/collection/map_entryset.png)