---
title: MySQL ACID 事务模型与隔离级别总结
date: 2019-03-20 12:39:16
updated:
tags: MySQL
---

# ACID 模型

维基百科关于 ACID 的定义：

> ACID 是[数据库事务](https://en.wikipedia.org/wiki/Database_transaction)的一组属性，旨在即使在发生错误、电源故障等情况下也能保证**数据有效性**。在数据库环境中，一系列满足 ACID 属性的数据库操作（可以视作对数据的单个逻辑操作）称为事务。例如，将资金从某个银行账户转账到另一个银行账户。

下面重点讨论 MySQL `InnoDB` 存储引擎如何与 ACID 模型进行交互：

## 原子性（Atomicity）

> 一个[事务](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_transaction)必须被视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部成功**提交**，要么全部失败**回滚**，对于一个事务来说，不可能只执行其中的一部分操作。

相关的 MySQL 功能包括：

* 事务的自动提交（`autocommit`）设置。
* `START TRANSACTION`、`COMMIT`、`ROLLBACK` 语句。

## 一致性（Consistency）

> 数据库总是从一个一致性的状态转换到另外一个一致性的状态，即使出现系统崩溃等异常情况。

相关的 MySQL 功能包括：

* `InnoDB` [双写缓冲区](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_doublewrite_buffer)。
* `InnoDB` [崩溃恢复](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_crash_recovery)。

## 隔离性（Isolation）

> 隔离性可以防止多个事务并发执行时由于交叉执行而导致的数据不一致问题。事务隔离分为不同级别，详见下述[隔离级别](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_isolation_level)。

相关的 MySQL 功能包括：

* 事务的自动提交（`autocommit`）设置。
* `SET TRANSACTION ISOLATION LEVEL` 语句。

## 持久性（Durability）

> 一旦事务提交，则其所做的修改会永久保存到数据库中。此时即使系统崩溃、修改的数据也不会丢失。

持久性是个有点模糊的概念，因为实际上持久性也分很多不同的级别。有些持久性策略能够提供非常强的安全保障，而有些则未必。而且不可能有能做到 100% 的持久性保证的策略，否则为何还要做数据库备份？

与持久性相关的 MySQL 功能比较多，这里不做讨论。

# 读现象问题

我们重点来关注下隔离性。隔离性可以防止多个事务并发执行时由于**交叉执行而导致的数据不一致问题**。因此如果不考虑隔离性，会引发如下问题：

## 脏读（Dirty reads）

一个事务能够看到其它事务尚未提交的修改。例如：

![脏读](/img/mysql/dirty_read.png)

## 不可重复读（Non-repeatable reads）

一个事务的两次查询返回不同的结果。例如：

![不可重复读](/img/mysql/non_repeatable_read.png)

有两种策略可以避免不可重复读：

* 采用共享锁（s-lock）或排它锁（x-lock），进行加锁读（Locking reads）。
* 采用多版本并发控制（MVCC），进行一致性非加锁读（Consistent Non-locking Reads）。

## 幻读（Phantom reads）

一个事务的两次查询返回不同的结果集。例如：

![幻读](/img/mysql/phantom_read.png)

# 隔离级别

通过提升事务的隔离级别（Isolation Level），可以逐一解决上述问题。所谓隔离级别，就是在数据库事务中，为保证多个事务**并发读写数据**的正确性而提出的定义，它并不是 MySQL 专有的概念，而是源于 [ANSI](https://en.wikipedia.org/wiki/American_National_Standards_Institute)/[ISO](https://en.wikipedia.org/wiki/International_Organization_for_Standardization) 制定的 [SQL-92](https://en.wikipedia.org/wiki/SQL-92) 标准。

每种关系型数据库都提供了各自特色的隔离级别实现，虽然在通常的隔离级别定义中是以锁为实现单元，但实际的实现千差万别。以最常见的 MySQL `InnoDB` 存储引擎为例，它是基于 [MVCC](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)（Multi-Versioning Concurrency Control）和锁的复合实现，性能较高。MySQL `InnoDB` 存储引擎的事务隔离级别及其解决问题如下：

| 隔离级别                            | 脏读<br/>（Dirty reads） | 不可重复读<br/>（Non-repeatable reads） | 幻读<br/>（Phantom reads）                            | SELECT 默认模式                                              |
| ----------------------------------- | ------------------------ | --------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| 读未提交<br/>（`READ UNCOMMITTED`） | √                        | √                                       | √                                                     |                                                              |
| 读已提交<br/>（`READ COMMITTED`）   | ×                        | √                                       | √                                                     | 使用一致性非加锁读（Consistent Non-locking Reads (MVCC)）<br/>总是使用**最新快照** |
| 可重复读<br/>（`REPEATABLE READ`）  | ×                        | ×                                       | ×（`InnoDB` 特有）<br/>使用 gap lock 或 next-key lock | 使用一致性非加锁读（Consistent Non-locking Reads (MVCC)）<br/>同一事务内总是使用**首次快照**，确保可重复读。 |
| 串行化<br/>（`SERIALIZABLE`）       | ×                        | ×                                       | ×<br/>使用 gap lock 或 next-key lock                  | 加共享锁读<br/>（S-Locking reads）                           |

## 读未提交（READ UNCOMMITTED）

一个事务能够看到其它事务尚未提交的修改，这是最低的隔离水平，允许**脏读**出现。

这个级别会导致很多问题，从性能上来说，也不会比其它级别好太多，但却缺乏其它级别的很多好处，实际应用中很少使用。

## 读已提交（READ COMMITTED）

事务能够看到的数据都是其它事务已经提交的修改，也就是保证不会看到任何中间性状态，因此不会出现脏读问题。但读已提交仍然是比较低的隔离级别，并不保证再次读取时能够获取同样的数据，也就是允许其它事务并发修改数据，允许不可重复读和幻读出现。

> Tips: 事务隔离级别越高，就越能保证数据的**完整性**和**一致性**，但同时对并发性能的影响也越大。通常，对于绝大多数的应用程序来说，在非 MySQL 数据库的情况下，可以优先考虑将数据库系统的隔离级别设置为**读已提交**，这能够在避免起码的脏读的同时，保证较好的并发性能。尽管这种事务隔离级别会导致不可重复读、幻读，但较为科学的做法是在可能出现这类问题的个别场合中，由应用程序**主动采取读锁或写锁**来进行事务控制。

MySQL 读已提交的默认行为如下：

* 同一事务中的[一致性读取（Consistent read）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_consistent_read)总是会设置和读取自己的最新[快照（snapshot）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_snapshot)，因此会产生不可重复读问题，因为其它事务可能会并发修改数据。

* 对于加锁读、`UPDATE`、`DELETE` 语句，`InnoDB` 仅锁定匹配的索引记录。由于禁用了 gap lock，因此会产生幻读问题，因为其它事务可以在[间隙（gap）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_gap)中插入新行。

  > gap:
  >
  > A place in an `InnoDB` **index** data structure where new values could be inserted. When you lock a set of rows with a statement such as `SELECT ... FOR UPDATE`, `InnoDB` can create locks that apply to the gaps as well as the actual values in the index.

  > gap lock:
  >
  > A **lock** on a **gap** between index records, or a lock on the gap before the first or after the last index record.

## 可重复读（REPEATABLE READ）

这是 MySQL `InnoDB` 存储引擎**默认的隔离级别**。

* 同一事务中的[一致性读取（Consistent read）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_consistent_read)总是会读取第一次读取时首次建立的[快照（snapshot）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_snapshot)。这意味着如果你在同一事务中发起多个普通（非加锁） `SELECT` 语句，其查询结果是相互一致的。一致性读取机制保证了同一事务中**可重复读**，避免了不可重复读问题，不管其它事务是否提交了 `INSERT`、`DELETE`、`UPDATE` 操作。如果想每次 `SELECT` 都返回最新快照，要么隔离级别降为 READ COMMITTED，要么使用加锁读。

* 对于加锁读、`UPDATE`、`DELETE` 语句，加锁行为取决于语句是使用具有唯一搜索条件的唯一索引还是范围搜索条件：

  * 对于具有唯一搜索条件的唯一索引， `InnoDB` 仅锁定匹配的索引记录。例如：

      ```sql
      -- 事务 T1 的 x-lock 会阻止其它事务加锁读或修改 id = 10 的记录
      SELECT * FROM parent WHERE id = 10 FOR UPDATE;
      -- 事务 T2 无法修改 id = 10 的记录，直到事务 T1 结束
      UPDATE parent SET name = 'Pete' WHERE id = 10;
      ```

  * 对于范围搜索条件，`InnoDB` 使用 [gap lock](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_gap_lock) 或 [next-key lock](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_next_key_lock) 锁定扫描到的索引范围， 以阻止其它会话插入被范围所覆盖的[间隙](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_gap)。这是 `InnoDB` 和其它一些数据库实现的不同，解决了可重复读级别下的幻读问题。例如：

      ```sql
      -- 事务 T1 的 gap lock 会阻止其它事务插入 id > 10 的记录
      SELECT * FROM parent WHERE id > 10 FOR UPDATE;
      -- 事务 T2 无法插入 id > 10 的新记录，直到事务 T1 结束
      INSERT INTO parent(id, name) VALUES(11, 'Pete');
      -- 事务 T2 可以插入 id <= 9 的新记录，无需等待事务 T1
      INSERT INTO parent(id, name) VALUES(9, 'Pete');
      ```

## 串行化（SERIALIZABLE）

并发事务之间的读写操作是串行化的，通常意味着读取需要获取共享锁（读锁），更新需要获取排他锁（写锁），如果 SQL 使用 `WHERE` 语句，还会获取 gap lock 和 next-key lock，可能导致大量的超时和锁争用的问题。

这是最高的隔离级别，实际应用中很少使用，只有在非常需要确保数据一致性而且可以接受没有并发的情况下，才会考虑。

# 参考

《高性能 MySQL》

https://en.wikipedia.org/wiki/Isolation_(database_systems)#Isolation_levels

https://en.wikipedia.org/wiki/Isolation_(database_systems)#Dirty_reads

https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_dirty_read

https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_non_repeatable_read

https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_phantom

https://dev.mysql.com/doc/refman/5.7/en/mysql-acid.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-transaction-model.html
