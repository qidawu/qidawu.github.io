---
title: Java 集合框架系列（三）并发实现总结
date: 2018-04-21 23:40:10
updated:
tags: Java
typora-root-url: ..
---

并发编程时，集合操作必须小心谨慎。Java SE 提供了两种并发编程方法：

* 方案一：为集合通用实现添加线程同步功能。`Collections` 工具类提供了称为[*同步包装器（synchronization wrappers）*](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedCollection-java.util.Collection-)的静态工厂方法可用于向许多非线程同步的集合添加同步行为。
* 方案二：使用并发集合。Java SE 提供了各种并发友好的集合接口和实现类。这些接口和类性能优于*同步包装器（synchronization wrappers）*，提供了并发编程中经常需要的各种功能。

# 同步包装器

`Collections` 工具类提供如下静态工厂方法：

```java
synchronizedCollection
synchronizedSet
synchronizedSortedSet
synchronizedNavigableSet
synchronizedList
synchronizedMap
synchronizedSortedMap
synchronizedNavigableMap
```

以 `synchronizedCollection` 静态方法为例，源码实现如下：

```java
    /**
     * Returns a synchronized (thread-safe) collection backed by the specified
     * collection.  In order to guarantee serial access, it is critical that
     * <strong>all</strong> access to the backing collection is accomplished
     * through the returned collection.<p>
     *
     * It is imperative that the user manually synchronize on the returned
     * collection when traversing it via {@link Iterator}, {@link Spliterator}
     * or {@link Stream}:
     * <pre>
     *  Collection c = Collections.synchronizedCollection(myCollection);
     *     ...
     *  synchronized (c) {
     *      Iterator i = c.iterator(); // Must be in the synchronized block
     *      while (i.hasNext())
     *         foo(i.next());
     *  }
     * </pre>
     * Failure to follow this advice may result in non-deterministic behavior.
     *
     * <p>The returned collection does <i>not</i> pass the {@code hashCode}
     * and {@code equals} operations through to the backing collection, but
     * relies on {@code Object}'s equals and hashCode methods.  This is
     * necessary to preserve the contracts of these operations in the case
     * that the backing collection is a set or a list.<p>
     *
     * The returned collection will be serializable if the specified collection
     * is serializable.
     *
     * @param  <T> the class of the objects in the collection
     * @param  c the collection to be "wrapped" in a synchronized collection.
     * @return a synchronized view of the specified collection.
     */
    public static <T> Collection<T> synchronizedCollection(Collection<T> c) {
        return new SynchronizedCollection<>(c);
    }
```

实现就是一行代码，返回一个静态内部类 `SynchronizedCollection`：

```java
    static class SynchronizedCollection<E> implements Collection<E>, Serializable {
        private static final long serialVersionUID = 3053995032091335093L;

        final Collection<E> c;  // Backing Collection
        final Object mutex;     // Object on which to synchronize

        SynchronizedCollection(Collection<E> c) {
            this.c = Objects.requireNonNull(c);
            mutex = this;
        }

        SynchronizedCollection(Collection<E> c, Object mutex) {
            this.c = Objects.requireNonNull(c);
            this.mutex = Objects.requireNonNull(mutex);
        }

        public int size() {
            synchronized (mutex) {return c.size();}
        }
        public boolean isEmpty() {
            synchronized (mutex) {return c.isEmpty();}
        }
        public boolean contains(Object o) {
            synchronized (mutex) {return c.contains(o);}
        }
        public Object[] toArray() {
            synchronized (mutex) {return c.toArray();}
        }
        public <T> T[] toArray(T[] a) {
            synchronized (mutex) {return c.toArray(a);}
        }

        public Iterator<E> iterator() {
            return c.iterator(); // Must be manually synched by user!
        }

        public boolean add(E e) {
            synchronized (mutex) {return c.add(e);}
        }
        public boolean remove(Object o) {
            synchronized (mutex) {return c.remove(o);}
        }

        public boolean containsAll(Collection<?> coll) {
            synchronized (mutex) {return c.containsAll(coll);}
        }
        public boolean addAll(Collection<? extends E> coll) {
            synchronized (mutex) {return c.addAll(coll);}
        }
        public boolean removeAll(Collection<?> coll) {
            synchronized (mutex) {return c.removeAll(coll);}
        }
        public boolean retainAll(Collection<?> coll) {
            synchronized (mutex) {return c.retainAll(coll);}
        }
        public void clear() {
            synchronized (mutex) {c.clear();}
        }
        public String toString() {
            synchronized (mutex) {return c.toString();}
        }
        // Override default methods in Collection
        @Override
        public void forEach(Consumer<? super E> consumer) {
            synchronized (mutex) {c.forEach(consumer);}
        }
        @Override
        public boolean removeIf(Predicate<? super E> filter) {
            synchronized (mutex) {return c.removeIf(filter);}
        }
        @Override
        public Spliterator<E> spliterator() {
            return c.spliterator(); // Must be manually synched by user!
        }
        @Override
        public Stream<E> stream() {
            return c.stream(); // Must be manually synched by user!
        }
        @Override
        public Stream<E> parallelStream() {
            return c.parallelStream(); // Must be manually synched by user!
        }
        private void writeObject(ObjectOutputStream s) throws IOException {
            synchronized (mutex) {s.defaultWriteObject();}
        }
    }
```

从源码可见，其实就是返回一个包装类，内部实现使用了 `synchronized` 关键字加 `mutex` 对象作为互斥锁，对被包装集合的所有增删改查方法加锁，相当暴力。

# 并发集合

并发集合作为 `java.util.concurrent` 包的一部分，提供的并发集合类如下：

![concurrent_collections](/img/java/collection/concurrent_collections.png)

## 队列

`Queue` 的常用方法差别如下：

|             | *Throws exception*                                           | *Returns special value*                                      |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Insert**  | [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#add-E-) | [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#offer-E-) |
| **Remove**  | [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#remove--) | [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#poll--) |
| **Examine** | [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) |

有两种并发队列实现，阻塞队列、并发队列（非阻塞式）：

![Queue_implementations](/img/java/collection/queue_impl.png)

### 阻塞队列

`BlockingQueue` 扩展自 `Queue` 接口，主要新增了两组接口（下表后两列），其常用方法差别如下：

|             | *Throws exception*                                           | *Returns Special value*                                      | *Times out*                                                  | *Blocks*                                                     |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Insert**  | [`add(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#add-E-) | [`offer(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#offer-E-) | [`offer(e, time, unit)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#offer-E-long-java.util.concurrent.TimeUnit-) | [`put(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#put-E-) |
| **Remove**  | [`remove()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#remove-java.lang.Object-) | [`poll()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#poll-long-java.util.concurrent.TimeUnit-) | [`poll(time, unit)`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#poll-long-java.util.concurrent.TimeUnit-) | [`take()`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html#take--) |
| **Examine** | [`element()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#element--) | [`peek()`](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html#peek--) | *not applicable*                                             | *not applicable*                                             |

`ThreadPoolExecutor` 目前使用了 `BlockingQueue` 的这些方法：

* 入队 `offer(e)`
* 出队 `poll(time, unit)` 或 `take()`

阻塞队列 `BlockingQueue` 的实现如下：

* 有界队列（数组实现）：
  * `ArrayBlockingQueue`，基于数组实现的有界阻塞队列。

* 无界队列（链表实现）：
  * `LinkedBlockingQueue`，基于链表实现的可选无界阻塞队列。默认用于 `Executors.newFixedThreadPool(...)` 和 `newSingleThreadExecutor(...)`。
  * `LinkedTransferQueue`
  * `PriorityBlockingQueue`，基于堆实现（底层是数组）的具有优先级的无界阻塞队列。
  * `DelayedQueue`

* 无容量队列：
  * `SynchronousQueue`，无容量阻塞队列，每个插入操作都必须等待另一个线程的移除操作，反之亦然。默认用于 `Executors.newCachedThreadPool(...)`。

![BlockingQueue](/img/java/collection/BlockingQueue_implementations.png)

阻塞双端队列 `BlockingDeque` 的实现如下：

![BlockingDeque_implementations](/img/java/collection/BlockingDeque_implementations.png)

### 并发队列（非阻塞）

`ConcurrentLinkedQueue`

`ConcurrentLinkedDeque`

## 并发散列表

`ConcurrentMap`

* `ConcurrentHashMap`

![ConcurrentHashMap](/img/java/collection/ConcurrentHashMap.png)

## 并发跳表

`ConcurrentSkipListSet`

`ConcurrentNavigableMap`

* `ConcurrentSkipListMap`

# 参考

https://blog.csdn.net/way2016/article/details/93380850

https://www.cnblogs.com/gujiande/p/9485493.html