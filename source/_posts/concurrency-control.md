---
title: 并发编程系列（一）两种并发控制机制的总结
date: 2019-02-18 12:05:04
updated:
tags: [并发编程, MySQL]
typora-root-url: ..
---

# 并发控制

计算机领域中，并发控制（Concurrency Control）是一种机制，它确保并发操作可以产生正确结果。

有两种常用的并发控制机制：

* 乐观并发控制（Optimistic Concurrency Control, OCC），又称为乐观锁（Optimistic Lock），最早是由孔祥重（H.T.Kung）教授提出的。
* 悲观并发控制（Pessimistic Concurrency Control, PCC），又称为悲观锁（Pessimistic Lock）。

这两种机制或者锁并不是 MySQL 或者数据库中独有的概念，而是并发编程的基本概念。

## 乐观并发控制

* 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，没有才能更新成功。
* 适用场景：

  * 它假设多用户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理**各自影响的那部分数据**。在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其它事务又修改了该数据。如果其它事务有更新的话，正在提交的事务会进行**回滚**。因此乐观并发控制多数用于数据争用不大、冲突较少的环境中。这种环境中，偶尔回滚事务的成本会低于读取数据时锁定数据的成本，因此可以获得比其它并发控制方法更高的吞吐量。
* 实现方式：

  * MVCC 多版本并发控制。

  * CAS（Compare And Set）：实现思路是在 `set` 的时候，加上初始状态的 `compare` 条件判断，只有初始状态不变时，才 `set` 成功。为了避免 ABA 问题（例如 CAS 过程中只简单进行“值”的校验，在有些情况下，“值”相同不会引入错误的业务逻辑（例如余额），但有些情况下，“值”虽然相同，却已经不是原来的数据了），CAS 不能只比对“值”，还**必须确保数据是原来的数据**，才能修改成功。实现方式是采用“数据版本”机制，例如通过版本号（version）、时间戳（update_time），来做乐观锁的判断条件，一个数据一个版本，版本变化，即使值相同，也不应该修改成功。

    * 例如，使用 Java 原子类 `AtomicStampedReference` 进行值与版本号的双重校验，而不是 `AtomicInteger`、`AtomicLong`、`AtomicBoolean`、`AtomicReference` 等等只基于值的校验。
    * 例如，MySQL：
    
        ```
        SELECT * FROM table_name WHERE condition=#condition#;
        UPDATE table_name SET name=#name#, version=version+1 WHERE version=#version#；
        ```

## 悲观并发控制

- 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，直到使用完毕才会解锁，这样别人想拿这个数据就会 block 住直到它拿到锁。

- 适用场景：

  - 悲观并发控制主要用于数据争用激烈的环境，以及发生并发冲突时使用锁保护数据的成本要低于回滚事务的成本的环境中。悲观锁大多数情况下依靠数据库的**锁机制**实现，以保证操作最大程度的独占性。但随之而来的就是数据库性能的大量开销，特别是对长事务而言，由于会阻塞其它事务导致其一直等待，**降低整体吞吐量**，这样的开销往往无法承受。而乐观锁机制则避免了长事务中的数据库开销。


- 使用场景：

### 原子性操作

集群部署环境下，面对并发请求，我们常常使用“一锁二判三更新”这套常规操作，其中第一步加锁是为了确保后两步操作的原子性，实现串行化访问临界资源，即同一时刻只能有一个线程独占性的访问临界资源（同步互斥访问），确保并发情况下临界资源的线程安全。

下面提供两种实现方案：

方案一 使用数据库行锁

- MySQL：在同一事务中先对资源加**排它锁（写锁）**，阻塞其它事务对同一资源的读写访问，然后在事务内进行代码判断以及资源更新提交，实现串行化访问资源：
  ```sql
  -- 共享锁（读锁）
  SELECT ... LOCK IN SHARE MODE;
  -- 排它锁（写锁）
  SELECT ... FOR UPDATE;
  ```

方案二 使用其它分布式锁，节约数据库资源（**推荐方案**）

- Redis：`SETNX` 互斥锁（mutex key）实现分布式锁，使用时注意设置锁的过期时间避免锁死，然后后台线程设置锁续命，解锁时只能由集群内同机器操作。
  
- Zookeeper：使用命令 `create -e -s` 创建临时+序号（`EPHEMERAL_SEQUENTIAL`）节点。

# MySQL 的并发控制

![并发控制总结](/img/mysql/concurrency_control.png)

MySQL `InnoDB` 存储引擎中，悲观[锁的类型](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html)还有很多种：

- Shared and Exclusive Locks（共享锁和排它锁）
- Intention Locks（意向锁）
- Record Locks（记录锁）
- Gap Locks（区间锁）
- Next-Key Locks
- Insert Intention Locks（插入意向锁）
- AUTO-INC Locks（自增锁）
- Predicate Locks for Spatial Indexes（空间索引谓词锁）

## 例子

这里举一个抽奖活动的例子，分别展示乐观锁和悲观锁的两种实现流程：

![抽奖活动例子](/img/mysql/example_of_concurrency_control.jpg)

# 参考

https://en.wikipedia.org/wiki/Concurrency_control

https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html

[《支付宝防并发方案之"一锁二判三更新"》](https://segmentfault.com/a/1190000011200547)

《高性能 MySQL》
