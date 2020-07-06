---
title: MySQL 一致性非加锁读（MVCC）总结
date: 2019-03-23 22:36:58
updated:
tags: MySQL
typora-root-url: ..
---

# 事务的隔离性

上文提到，数据库的事务隔离性，主要解决以下问题：

* 防止多个事务并发执行时由于交叉执行而导致的数据不一致问题。
* 解决同一事务内的多次相同查询，数据不一致问题。

有哪些数据不一致的情况？

* 脏读
* 不可重复读
* 幻读

为了数据不一致问题，引入了四个隔离级别，随着隔离级别的提升，可以解决上述更多情况。它们所使用的 `SELECT` 模式分别如下：

| 隔离级别                        | SELECT 默认模式                                              | 备注                                                         |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 读未提交<br/>`READ UNCOMMITTED` | /                                                            |                                                              |
| 读已提交<br/>`READ COMMITTED`   | 使用一致性非加锁读（Consistent Non-locking Reads）<br/>总是使用**最新快照** |                                                              |
| 可重复读<br/>`REPEATABLE READ`  | 使用一致性非加锁读（Consistent Non-locking Reads）<br/>同一事务内总是使用**首次快照**，确保可重复读。 | 一致性读取不会在它访问的数据上加任何锁，因此其它事务可以自由地同时修改那些数据，同一份数据在 undo log 会存在**多份历史版本**。（即通过多版本并发控制（MVCC）实现可重复读） |
| 串行化<br/>`SERIALIZABLE`       | 加共享锁读<br/>（S-Locking reads）                           | 加锁读会给数据加共享锁，其它事务读取时可以继续加共享锁，但修改会阻塞等待以获取排它锁，保证读写的串行化，因此同一份数据只存在**一份当前版本**。（即通过读写锁实现可重复读） |

# MySQL 可重复读实现

下面重点看下 MySQL InnoDB 如何实现可重复读这个隔离级别。它使用了一致性非加锁读（Consistent Non-locking Reads）实现多版本并发控制（MVCC），这种方法不会在它访问的表上设置任何锁，因此其它事务可以自由地同时修改那些表，并发性能高。

## 示例

下图展示了两个事务并发执行时，最终会出现的五种情况：

![consistent read examples](/img/mysql/consistent-read-examples.png)

即：

> 事务的可重复读的能力是怎么实现的？
>
> 可重复读的核心就是一致性读（consistent read）；而事务更新数据的时候，只能用当前读。如果当前的记录的行锁被其他事务占用的话，就需要进入锁等待。





数据库快照适用于同一事务内的 `SELECT` 语句，而不一定适用于 DML 语句。不同事务间的增删改操作还是会相互影响，例如：

* 尽管事务 A 创建一致性视图时查不到 `xyz` 记录，但如果此后其它事务插入了 `xyz` 记录并提交事务，事务 A 仍然可以将它们删除：

  ```sql
  SELECT COUNT(c1) FROM t1 WHERE c1 = 'xyz';
  -- Returns 0: no rows match.
  DELETE FROM t1 WHERE c1 = 'xyz';
  -- Deletes several rows recently committed by other transaction.
  ```

* 尽管事务 A 创建一致性视图时查不到 `abc` 记录，但如果此后其它事务插入了 `abc` 记录并提交事务，事务 A 仍然可以修改这些记录，并看到本事务内的修改：

  ```sql
  SELECT COUNT(c2) FROM t1 WHERE c2 = 'abc';
  -- Returns 0: no rows match.
  UPDATE t1 SET c2 = 'cba' WHERE c2 = 'abc';
  -- Affects 10 rows: another txn just committed 10 rows with 'abc' values.
  SELECT COUNT(c2) FROM t1 WHERE c2 = 'cba';
  -- Returns 10: this txn can now see the rows it just updated.
  ```

## 实现原理

### Undo Log

### Read View

> 在实现上， InnoDB 为每个事务构造了一个数组，用来保存这个事务启动瞬间，当前正在“活跃”的所有事务 ID。“活跃”指的就是，启动了但还没提交。
>
> 数组里面事务 ID 的最小值记为低水位，当前系统里面已经创建过的事务 ID 的最大值加 1 记为高水位。
>
> 这个视图数组和高水位，就组成了当前事务的一致性视图（read-view）。
>
> 这个视图数组把所有的 row trx_id 分成了几种不同的情况。如下图：

![consistent-read-view](/img/mysql/consistent-read-view.png)

已下表事务为例，一致性视图为：`[100,103,104,105],106`，其中低水位为 `100`，高水位为 `106`。这些事务分布如上图。

| row trx_id | committed? | remark      |
| ---------- | ---------- | ----------- |
| 100        | N          |             |
| 101        | Y          |             |
| 102        | Y          |             |
| 103        | N          |             |
| 104        | N          |             |
| 105        | N          | current trx |

对于当前事务 ID `105`，根据以下流程图，就只能看到已提交事务 `1-99`, `101`, `102`

![consistent read process](/img/mysql/consistent-read-process.png)

**数据版本的可见性规则，就是基于数据的 row trx_id 和这个一致性视图的对比结果得到的。**假如事务 ID `100-104` 依次修改了同一份数据（如上图右），虽然数据当前版本为 `104`，但对于当前事务 ID `105` 来说，也只能看到版本链上事务 ID `102` 提交的数据版本。

## 如何查看最新快照

如果要查看最新快照，可以通过以下三个方法：

* 使用 `READ COMMITTED` 隔离级别
* 提交当前事务并发起新查询，刷新时间点
* 使用加锁读（读锁或写锁）

下例展示了第二种方法：

```
             Session A              Session B

           START TRANSACTION;     START TRANSACTION;
time
|          SELECT * FROM t;
|          empty set
|                                 INSERT INTO t VALUES (1, 2);
|
v          SELECT * FROM t;
           empty set
                                  COMMIT;

           SELECT * FROM t;
           empty set

           COMMIT;

           SELECT * FROM t;
           ---------------------
           |    1    |    2    |
           ---------------------
```

## 不适用的情况

一致性读取不适用于某些 DDL 语句：

* `DROP TABLE`
* `ALTER TABLE`

# 参考

《高性能 MySQL》

https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html

https://time.geekbang.org/column/article/70562