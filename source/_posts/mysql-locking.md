---
title: MySQL 加锁读机制总结
date: 2018-10-20 22:05:04
updated:
tags: MySQL
typora-root-url: ..
---

MySQL 支持两种读机制：

* [一致性非加锁读（Consistent Non-locking Reads (MVCC)）](https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html)
* [加锁读（Locking Reads）](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)

# 加锁读机制

`InnoDB` 支持两种类型的 [加锁读（Locking Reads）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_locking_read)，为事务操作提供额外的**安全性**：

* [共享锁（Shared Lock, S-Lock）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_shared_lock)，也叫读锁（Read Lock）
  * 语法：`SELECT ... LOCK IN SHARE MODE ` or `SELECT ... FOR SHARE` in MySQL 8.0.1，在检索行上设置共享锁（s-lock）
  * 其它事务允许读取检索行，但不允许更新或删除，更新或删除会一直阻塞等待，直到该事务结束。
* [排它锁（Exclusive Lock, X-Lock）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_exclusive_lock)，也叫写锁（Write Lock）
  * 语法：`SELECT ... FOR UPDATE` 在检索行上设置排它锁（x-lock）
  * 其它事务不允许更新或删除
  * 不允许加共享锁读取 `SELECT ... LOCK IN SHARE MODE`
  * 如果事务隔离级别为 `SERIALIZABLE`，不允许读取（因为该级别的读取默认需要获得共享读锁）
  * 上述操作将一直阻塞等待，直到该事务结束。

共享锁和排它锁之间存在冲突的四种情况总结如下：

|           | T1 持有共享锁 | T1 持有排它锁 |
| ----------------- | ------------- | ------------- |
| **T2 获取共享锁** | 兼容          | 冲突          |
| **T2 获取排它锁** | 冲突          | 冲突          |

下面进一步分析共享锁和排它锁：

## 共享锁（读锁）

共享锁是共享性的，或者说是相互不阻塞的。持有该锁的多个事务允许同时读取同一个资源，而互不干扰。

举个例子，如果事务 `T1` 持有对行 `r` 的共享锁，那么来自另一个事务 `T2` 的锁请求，将按如下两种方式处理：

- `T2` 的共享锁请求能够立即授予。其结果是，`T1` 和 `T2` 都持有对行 `r` 的共享锁。
- `T2` 的排它锁请求不被授予。

## 排它锁（写锁）

排它锁是排它性的，也就是说一个排它锁会阻塞其它的共享锁和排它锁，这是出于安全策略的考虑，只有这样，才能确保在给定的时间里，有且只有一个持有该锁的事务执行更新或删除操作，并防止其它事务读取正在操作的同一资源。

举个例子，如果事务 `T1` 持有对行 `r` 的排它锁，那么来自另一个事务 `T2` 的**任一锁请求都不被授予**。相反，事务 `T2` 必须等待事务 `T1` 直到其释放对行 `r` 的锁定。

# 锁定方式

大多数时候，MySQL 锁的内部管理都是透明的，其表现如下：

- `SELECT` 在 `InnoDB` 的读已提交（READ COMMITTED）、可重复读（REPEATABLE READ）这两种事务隔离级别下，默认采用[一致性非加锁读取](https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html)，因此**无需加锁即可读取所需数据**。
- 如果需要使用[加锁读](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)提升数据安全性，实现悲观并发控制，可采用共享锁（`LOCK IN SHARE MODE`）或排它锁（`FOR UDPATE`）进行显式锁定。
- `UPDATE`、`DELETE` 默认采用排它锁，隐式锁定。

总结如下：

| 语句                            | 锁的类型                                            | 锁定方式 |
| ------------------------------- | --------------------------------------------------- | -------- |
| `SELECT ... FROM`               | 如果事务隔离为 SERIALIZABLE，使用共享锁。否则无锁。 | 隐式锁定 |
| `SELECT ... LOCK IN SHARE MODE` | 共享锁（shared next-key lock）                      | 显式锁定 |
| `SELECT ... FOR UDPATE`         | 排它锁（exclusive next-key lock）                   | 显式锁定 |
| `UPDATE ... WHERE ...`          | 排它锁（exclusive next-key lock）                   | 隐式锁定 |
| `DELETE FROM ... WHERE ...`     | 排它锁（exclusive next-key lock）                   | 隐式锁定 |
| `INSERT`                        | 排它锁（exclusive index-record lock）               | 隐式锁定 |

## 隐式锁定

`InnoDB` 采用的是两阶段锁定协议（Two-phase locking protocol）。在事务执行过程中，随时都可以执行锁定，锁只有在执行 `COMMIT` 或者 `ROLLBACK` 的时候才会释放，并且所有的锁是在同一时刻被释放。`InnoDB` 会根据隔离级别在需要的时候自动加锁，例如下列操作：

- `UPDATE`、`DELETE` 

## 显式锁定

`InnoDB` 也支持通过特定语句进行显式锁定，这些语句不属于 SQL 规范：

- `SELECT ... LOCK IN SHARE MODE`（共享锁）
- `SELECT ... FOR UDPATE`（排它锁）

MySQL 也支持 `LOCK TABLES` 和 `UNLOCK TABLE` 语句，这是在服务器层实现的，和存储引擎无关。它们有自己的用途，但并不能替代事务。如果应用需要用到事务，还是应该选择事务型存储引擎。

经常可以发现，应用已经将表从 `MyISAM` 转换到 `InnoDB`，但还是显示地使用 `LOCK TABLE` 语句。这不但没有必要，还会严重影响性能，实际上 `InnoDB` 的行级锁工作得更好。

# 例子

这里举个例子，有一张 parent 和 child 表：

```sql
-- parnet 表
CREATE TABLE `parent` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;

-- child 表，其中 parent_id 字段外键关联 parent 表的 id 主键
CREATE TABLE `child` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `parent_id` bigint(20) NOT NULL,
  `name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_parent_id` (`parent_id`),
  CONSTRAINT `fk_parent_id` FOREIGN KEY (`parent_id`) REFERENCES `parent` (`id`)
) ENGINE=InnoDB;
```

如果在同一事务中先查询、后插入或更新相关数据，常规的 `SELECT` 语句无法得到足够保护。因为在此期间其它事务可能对同一资源进行更新或删除。例如：

```sql
--开启事务 T1
START TRANSACTION;
--为变量@id赋值
set @id=0;
SELECT @id:=id FROM parent WHERE name = 'Heikki';

--在此期间，某个事务 T2 成功删除了同一资源
DELETE FROM parent WHERE name = 'Heikki';

--事务 T1 插入失败：外键关联错误
INSERT INTO child(parend_id, name) VALUES(@id, 'Baby');
1452 - Cannot add or update a child row: a foreign key constraint fails (`test`.`child`, CONSTRAINT `fk_parent_id` FOREIGN KEY (`parent_id`) REFERENCES `parent` (`id`))
```

下面分别看下如何用共享锁和排它锁解决这个问题：

## LOCK IN SHARE MODE

```sql
--开启事务 T1
START TRANSACTION;
select * from child where parent_id = 2;
+----+-----------+-------+
| id | parent_id | name  |
+----+-----------+-------+
|  1 |         2 | Baby  |
|  2 |         2 | Baby5 |
+----+-----------+-------+
2 rows in set

--在此期间，某个事务 T2 能够成功删除同一资源
delete from child where id = 1;
Query OK, 1 row affected

--事务 T1 如果继续使用一致性非加锁读，将会得到第一次读取时的快照，因为 InnoDB 当前隔离级别为 RR
select * from child where parent_id = 2;
+----+-----------+-------+
| id | parent_id | name  |
+----+-----------+-------+
|  1 |         2 | Baby  |
|  2 |         2 | Baby5 |
+----+-----------+-------+
2 rows in set

--事务 T1 如果使用加锁读，将会得到最新快照。同时事务 T1 获取该行的共享锁，其它任何事务都只能读、不能写该行，直到事务 T1 结束，释放共享锁
select * from child where parent_id = 2 lock in share mode;
+----+-----------+-------+
| id | parent_id | name  |
+----+-----------+-------+
|  2 |         2 | Baby5 |
+----+-----------+-------+
1 row in set

--在此期间，事务 T3 可以删除未被锁定的行
delete from child where id = 3;
Query OK, 1 row affected

--在此期间，事务 T3 无法删除带锁的行。因为它无法获取该行的排它锁，因此会阻塞直到事务 T1 解锁该行。如果等待超时，则事务回滚
delete from child where id = 2
1205 - Lock wait timeout exceeded; try restarting transaction

--事务 T1 提交，释放共享锁
commit;

--事务 T3 如果没有超时，则操作成功
Query OK, 1 row affected
```

## FOR UPDATE

```sql
--开启事务 T1
START TRANSACTION;
--事务 T1 获取该行的排它锁
select * from child where parent_id = 2 for update;

--在此期间，事务 T2 可以非加锁读，因为无需先获取该行的锁
select * from child where parent_id = 2;
--也可以加共享锁读非锁定行
select * from child where parent_id = 3 lock in share mode;
--但无法加共享锁读锁定行
select * from child where parent_id = 2 lock in share mode;
--也无法获取排它锁进行修改
update child set name = 'Hello' where parent_id = 2;
```

————分割线————

可见，通过共享锁和排它锁都能解决这个问题。下例演示通过 `SELECT ... LOCK IN SHARE MODE ` 设置共享锁解决开头那个问题：

```sql
--开启事务 T1
START TRANSACTION;
--为变量@id赋值
set @id=0;
SELECT @id:=id FROM parent WHERE NAME = 'Heikki' LOCK IN SHARE MODE;

--在此期间，某个事务 T2 无法删除同一资源。因为 T2 会一直等待，直到 T1 事务完成，所有数据都处于一致状态，并释放共享锁之后，T2 才能获取排它锁，并对数据进行修改
DELETE FROM parent WHERE name = 'Heikki';

--事务 T1 插入成功
INSERT INTO child(parend_id, name) VALUES(@id, 'Baby');
--提交事务 T1，写库
COMMIT;
```

在 `T1` 成功提交事务并释放共享锁之后，`T2 ` 获得排它锁。但由于 `T1` 在 `child` 表中写入了一条对 `parent` 表的外键关联记录，所以 `T2` 删除失败：

```sql
1451 - Cannot delete or update a parent row: a foreign key constraint fails (`test`.`child`, CONSTRAINT `fk_parent_id` FOREIGN KEY (`parent_id`) REFERENCES `parent` (`id`))
```

最后，提几个注意点：

* 只有在通过以下方式之一禁用[自动提交（autocommit）](https://dev.mysql.com/doc/refman/5.7/en/innodb-autocommit-commit-rollback.html)时，才能加锁读：
  * 通过 [`START TRANSACTION`](https://dev.mysql.com/doc/refman/5.7/en/commit.html) 语句，显式开启事务；
  * 通过设置 [`autocommit`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_autocommit)为 `0`，显式关闭自动提交。
* 加锁读有可能产生**死锁**，具体取决于事务的隔离级别。

# 参考

《高性能 MySQL》

https://en.wikipedia.org/wiki/Two-phase_locking

https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html

https://blog.csdn.net/claram/article/details/54023216