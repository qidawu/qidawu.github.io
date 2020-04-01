---
title: MySQL MVCC 一致性非加锁读总结
date: 2019-02-19 22:36:58
updated:
tags: MySQL
typora-root-url: ..
---

一致性读取是指 `InnoDB` 存储引擎使用多个版本来为 `SELECT` 查询呈现数据库某个时间点的快照。该查询只能看到在该时间点之前提交的事务、或本事务内所做的修改，而无法看到之后提交或其它未提交的事务所做的修改。

如果要查看最新快照，可以通过以下三个方法：

* 提交当前事务并发起新查询，刷新时间点
* 使用 `READ COMMITTED` 隔离级别
* 使用加锁读（读锁或写锁）

下例展示了第一种方法：

```
             Session A              Session B

           SET autocommit=0;      SET autocommit=0;
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

不同隔离级别下的情况：

| 隔离级别                            | SELECT 默认模式                                              |
| ----------------------------------- | ------------------------------------------------------------ |
| 读未提交<br/>（`READ UNCOMMITTED`） | /                                                            |
| 读已提交<br/>（`READ COMMITTED`）   | 一致性非加锁读，总是使用最新快照<br/>（Consistent Non-locking Reads (MVCC)） |
| 可重复读<br/>（`REPEATABLE READ`）  | 一致性非加锁读，同一事务内总是使用首次快照<br/>（Consistent Non-locking Reads (MVCC)） |
| 串行化<br/>（`SERIALIZABLE`）       | 加共享锁读<br/>（S-Locking reads）                           |

数据库快照适用于同一事务内的 `SELECT` 语句，而不一定适用于 DML 语句。不同事务间的增删改操作会相互影响，例如：

```sql
SELECT COUNT(c1) FROM t1 WHERE c1 = 'xyz';
-- Returns 0: no rows match.
DELETE FROM t1 WHERE c1 = 'xyz';
-- Deletes several rows recently committed by other transaction.

SELECT COUNT(c2) FROM t1 WHERE c2 = 'abc';
-- Returns 0: no rows match.
UPDATE t1 SET c2 = 'cba' WHERE c2 = 'abc';
-- Affects 10 rows: another txn just committed 10 rows with 'abc' values.
SELECT COUNT(c2) FROM t1 WHERE c2 = 'cba';
-- Returns 10: this txn can now see the rows it just updated.
```

一致性读取不会在它访问的表上设置任何锁，因此与此同时其它会话可以自由地修改那些表。

一致性读取不适用于某些 DDL 语句：

* `DROP TABLE`
* `ALTER TABLE`

# 参考

《高性能 MySQL》

https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html