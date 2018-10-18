---
title: MySQL 锁机制总结
date: 2018-10-20 22:05:04
updated:
tags: MySQL
---

# 锁的类型

MySQL `InnoDB` 存储引擎中，[锁的类型](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html)有很多种：

- Shared and Exclusive Locks（共享锁和排他锁）
- Intention Locks（意向锁）
- Record Locks（记录锁）
- Gap Locks（区间锁）
- Next-Key Locks
- Insert Intention Locks（插入意向锁）
- AUTO-INC Locks（自增锁）
- Predicate Locks for Spatial Indexes（空间索引谓词锁）

这里只重点讨论其中一种—— [共享锁（Shared Lock, S-Lock）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_shared_lock) 和 [排它锁（Exclusive Lock, X-Lock）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_exclusive_lock)，也叫读锁（Read Lock）和写锁（Write Lock）。

## 共享锁

共享锁是共享性的，或者说是相互不阻塞的。持有该锁的多个事务允许同时读取同一个资源，而互不干扰。

举个例子，如果事务 `T1` 持有对行 `r` 的共享锁，那么来自另一个事务 `T2` 的锁请求，将按如下两种方式处理：

- `T2` 的共享锁请求能够立即授予。其结果是，`T1` 和 `T2` 都持有对行 `r` 的共享锁。
- `T2` 的排它锁请求不被授予。

## 排它锁

排它锁是排它性的，也就是说一个排它锁会阻塞其它的共享锁和排它锁，这是出于安全策略的考虑，只有这样，才能确保在给定的时间里，有且只有一个持有该锁的事务执行更新或删除操作，并防止其它事务读取正在操作的同一资源。

举个例子，如果事务 `T1` 持有对行 `r` 的排它锁，那么来自另一个事务 `T2` 的**任一锁请求都不被授予**。相反，事务 `T2` 必须等待事务 `T1` 直到其释放对行 `r` 的锁定。



四种情况的总结如下：

|           | T1 持有共享锁 | T1 持有排它锁 |
| ----------------- | ------------- | ------------- |
| **T2 获取共享锁** | 兼容          | 冲突          |
| **T2 获取排它锁** | 冲突          | 冲突          |

大多数时候，MySQL 锁的内部管理都是透明的，其表现如下：

- `SELECT` 在 `InnoDB` 在读已提交、可重复读的事务隔离级别下，默认采用[一致性非加锁读取](https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html)，因此**无需加锁即可读取所需数据**。如果需要使用[加锁读](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)提升数据安全性，实现悲观并发控制，可采用排它性的共享锁（`LOCK IN SHARE MODE`）或排它锁（`FOR UDPATE`），参考《[MySQL 加锁读机制总结](/2018/10/21/mysql-locking-reads/)》。
- `UPDATE`、`DELETE` 默认采用排它锁。

# 锁的粒度

所谓的锁策略，就是在锁的开销和数据的安全性之间寻求平衡，这种平衡当然也会影响到性能。

> 一种提高共享资源并发性的方式就是让锁定对象更有**选择性**。尽量只锁定需要修改的部分数据，而不是所有资源。更理想的方式是，只对会修改的数据片进行精确的锁定。任何时候，在给定的资源上，锁定的数据量越少，则系统的并发程度越高，只要相互之间不发生冲突即可。
>
> 问题是加锁也需要消耗资源。锁的各种操作，包括获得锁、检查锁是否已经解除、释放锁等，都会增加系统的开销。如果系统花费大量的时间来管理锁，而不是存取数据，那么系统的性能可能会因此受到影响。

## 表锁

表锁（Table Lock）是 MySQL 中最基本的锁策略，并且是开销最小的策略。

## 行锁

行锁（Row Lock）可以最大程度的支持并发处理，同时也带来了最大的锁开销。行锁只在存储引擎层实现，而不在 MySQL 服务器层。`InnoDB` 的锁粒度就是到行级别的。

# 锁定方式

## 隐式锁定

`InnoDB` 采用的是两阶段锁定协议（Two-phase locking protocol）。在事务执行过程中，随时都可以执行锁定，锁只有在执行 `COMMIT` 或者 `ROLLBACK` 的时候才会释放，并且所有的锁是在同一时刻被释放。`InnoDB` 会根据隔离级别在需要的时候自动加锁，例如下列操作：

- `UPDATE`、`DELETE` 

## 显式锁定

`InnoDB` 也支持通过特定语句进行显式锁定，这些语句不属于 SQL 规范：

- `SELECT ... LOCK IN SHARE MODE`（共享锁）
- `SELECT ... FOR UDPATE`（排它锁）

MySQL 也支持 `LOCK TABLES` 和 `UNLOCK TABLE` 语句，这是在服务器层实现的，和存储引擎无关。它们有自己的用途，但并不能替代事务。如果应用需要用到事务，还是应该选择事务型存储引擎。

经常可以发现，应用已经将表从 `MyISAM` 转换到 `InnoDB`，但还是显示地使用 `LOCK TABLE` 语句。这不但没有必要，还会严重影响性能，实际上 `InnoDB` 的行级锁工作得更好。

# 死锁问题

死锁是指两个或多个事务在同一资源上相互占用，并请求锁定对方占用的资源，从而导致恶性循环的现象。

为了解决这种问题，数据库系统实现了各种**死锁检测**和**死锁超时**机制。越复杂的系统，比如 `InnoDB` 存储引擎，越能检测到死锁的循环依赖，并立即返回一个错误。这种解决方式很有效，否则死锁会导致出现非常慢的查询。还有一种解决方式，就是当查询的时间达到锁等待超时的设定后放弃锁请求，例如：

```sql
1205 - Lock wait timeout exceeded; try restarting transaction
```

`InnoDB` 目前处理死锁的方法是，**将持有最少行级排它锁的事务进行回滚**，这是相对比较简单的死锁回滚算法。

锁的行为和顺序是和存储引擎相关的。以同样的顺序执行语句，有些存储引擎会产生死锁，有些则不会。死锁的产生有双重原因：

1. 有些是因为真正的数据冲突，这种情况通常很难避免。
2. 有些则完全是由于存储引擎的实现方式导致的。

**死锁发生以后，只有部分或者完全回滚其中一个事务，才能打破死锁。**对于事务型的系统，这是无法避免的，所以应用程序在设计时必须考虑如何处理死锁。大多数情况下只需要重新执行因死锁回滚的事务即可。

# 总结

| 语句                            | 锁的类型                                            | 锁定方式 |
| ------------------------------- | --------------------------------------------------- | -------- |
| `SELECT ... FROM`               | 如果事务隔离为 SERIALIZABLE，使用共享锁。否则无锁。 | 隐式锁定 |
| `SELECT ... LOCK IN SHARE MODE` | 共享锁（shared next-key lock）                      | 显式锁定 |
| `SELECT ... FOR UDPATE`         | 排它锁（exclusive next-key lock）                   | 显式锁定 |
| `UPDATE ... WHERE ...`          | 排它锁（exclusive next-key lock）                   | 隐式锁定 |
| `DELETE FROM ... WHERE ...`     | 排它锁（exclusive next-key lock）                   | 隐式锁定 |
| `INSERT`                        | 排它锁（exclusive index-record lock）               | 隐式锁定 |

![并发控制总结](/img/mysql/concurrency_control.png)

# 参考

《高性能 MySQL》

https://en.wikipedia.org/wiki/Two-phase_locking

https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-deadlocks.html

[支付宝防并发方案之"一锁二判三更新"](https://segmentfault.com/a/1190000011200547)