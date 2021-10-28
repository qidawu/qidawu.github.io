---
title: Java 集合框架系列（六）线性表之 Queue 实现总结
date: 2018-05-12 23:40:10
updated:
tags: [Java, 数据结构, 并发编程]
typora-root-url: ..
---

本文介绍一种运算受限的线性表 —— 队列在 Java 中的实现与使用：

![线性结构](/img/data-structure/list/list.jpg)

# 队列接口

`Queue` 接口的继承关系及提供的方法如下：

![Queue 接口提供的方法](/img/java/collection/methods_of_collection.png)

## 入队/出队/队头

`Queue` 接口提供的六个方法差别如下：

|             | *Throws exception*                                           | *Returns special value*                                      |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Insert**  | [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#add-E-) | [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#offer-E-) |
| **Remove**  | [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#remove--) | [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#poll--) |
| **Examine** | [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) |

## 遍历队列

参考：《[Iterate over a Queue in Java](https://www.techiedelight.com/iterate-through-queue-java/)》

# 队列实现

Java 提供的队列实现如下图：

![collection_impl](/img/java/collection/collection_impl.png)

## 顺序队列

使用如下：

```java
// 单端顺序队列
Queue<Integer> queue = new ArrayDeque<>(10);

// 双端顺序队列
Deque<Integer> deque = new ArrayDeque<>(10);
```

`ArrayDeque` 底层存储结构为**一维数组**，默认队列容量为 16。其成员变量定义如下：

```java
    /**
     * The array in which the elements of the deque are stored.
     * The capacity of the deque is the length of this array, which is
     * always a power of two. The array is never allowed to become
     * full, except transiently within an addX method where it is
     * resized (see doubleCapacity) immediately upon becoming full,
     * thus avoiding head and tail wrapping around to equal each
     * other.  We also guarantee that all array cells not holding
     * deque elements are always null.
     */
    transient Object[] elements; // non-private to simplify nested class access

    /**
     * The index of the element at the head of the deque (which is the
     * element that would be removed by remove() or pop()); or an
     * arbitrary number equal to tail if the deque is empty.
     */
    transient int head;

    /**
     * The index at which the next element would be added to the tail
     * of the deque (via addLast(E), add(E), or push(E)).
     */
    transient int tail;
```

在反复入队、出队时，为了解决了一维数组的“空间浪费”及“假溢出”问题，有两种解决方案：

* 在出队时，将剩余数组元素全部往前挪动一个位置，但这个动作的时间复杂度 `O(n)`，影响性能。
* 使用循环队列。

`ArrayDeque` 是一个**循环队列**。当进行出队、入队操作时，并不是简单是 `head++`、`tail++`，而是自增后进行取模运算，确保 `head`、`tail` 在容量范围内进行反复循环。其实现如下图：

![入队/出队](/img/java/collection/queue/enqueue_dequeue.png)

`ArrayDeque` 是一个**无界队列**。当队满时，会进行双倍空间的**动态扩容**（底层实现为新建一个双倍容量的新数组，并使用 `System#arraycopy` 方法进行数组拷贝）。源码实现如下：

```java
    /**
     * Doubles the capacity of this deque.  Call only when full, i.e.,
     * when head and tail have wrapped around to become equal.
     */
    private void doubleCapacity() {
        assert head == tail;
        int p = head;
        int n = elements.length;
        int r = n - p; // number of elements to the right of p
        int newCapacity = n << 1;
        if (newCapacity < 0)
            throw new IllegalStateException("Sorry, deque too big");
        Object[] a = new Object[newCapacity];
        System.arraycopy(elements, p, a, 0, r);
        System.arraycopy(elements, 0, a, r, p);
        elements = a;
        head = 0;
        tail = n;
    }
```

这里我用 C 语言自己实现了一个简单的循环队列，代码如下：

https://github.com/qidawu/cpp-data-structure/blob/main/src/cycleQueue.cpp

实现过程中，有一些注意点：

![队列实现注意点](/img/data-structure/list/queue_impl.png)

## 链式队列

使用如下。链式队列也是一个**无界队列**：

```java
// 单端链式队列
Queue<Integer> queue = new LinkedList<>();

// 双端链式队列
Deque<Integer> deque = new LinkedList<>();
```

## 阻塞队列

并发队列有两种实现方式：

* 阻塞队列（`BlockingQueue`）
* 并发队列（非阻塞实现）

![Queue_implementations](/img/java/collection/queue/queue_impl.png)

下面重点看下 `BlockingQueue`，其扩展自 `Queue` 接口，主要新增了两组接口（下表后两列），其常用方法差别如下表：

|             | *Throws exception*                                           | *Returns Special value*                                      | *Times out*                                                  | *Blocks*                                                     |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Insert**  | [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#add-E-) | [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#offer-E-) | [`offer(e, time, unit)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#offer-E-long-java.util.concurrent.TimeUnit-) | [`put(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#put-E-) |
| **Remove**  | [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#remove-java.lang.Object-) | [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#poll-long-java.util.concurrent.TimeUnit-) | [`poll(time, unit)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#poll-long-java.util.concurrent.TimeUnit-) | [`take()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#take--) |
| **Examine** | [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) | *not applicable*                                             | *not applicable*                                             |

而线程池 `ThreadPoolExecutor` 目前就使用了 `BlockingQueue` 的这些方法：

* 入队 `offer(e)`
* 出队 `poll(time, unit)` 或 `take()`

### 阻塞队列实现

阻塞队列 `BlockingQueue` 的实现如下图：

![BlockingQueue](/img/java/collection/queue/BlockingQueue_implementations.png)

* 有界队列（数组实现）：
  * `ArrayBlockingQueue`，基于数组实现的有界阻塞队列。

* 无界队列（链表实现）：
  * `LinkedBlockingQueue`，基于链表实现的可选无界阻塞队列。默认用于 `Executors.newFixedThreadPool(...)` 和 `newSingleThreadExecutor(...)`。
  * `LinkedTransferQueue`
  * `PriorityBlockingQueue`，基于堆实现（底层是数组）的具有优先级的无界阻塞队列。
  * `DelayedQueue`

* 无容量队列：
  * `SynchronousQueue`，无容量阻塞队列，每个插入操作都必须等待另一个线程的移除操作，反之亦然。默认用于 `Executors.newCachedThreadPool(...)`。

阻塞双端队列 `BlockingDeque` 的实现如下图：

![BlockingDeque_implementations](/img/java/collection/queue/BlockingDeque_implementations.png)

# 参考

《[[java 队列]——ArrayBlockingQueue](https://blog.csdn.net/way2016/article/details/93380850)》

《[ConcurrentLinkedQueue 和 LinkedBlockingQueue 区别](https://www.cnblogs.com/gujiande/p/9485493.html)

