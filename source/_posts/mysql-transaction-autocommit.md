---
title: MySQL 事务自动提交机制总结
date: 2018-10-27 23:39:16
updated:
tags: MySQL
---

# 事务的自动提交机制

在 `InnoDB`，所有用户活动都发生在事务中。

`InnoDB` 默认采用事务[自动提交](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_autocommit)（`autocommit`）机制。也就是说，如果不是显式开启一个事务，则每条 SQL 语句都**形成独立事务**。如果该语句执行后没有返回错误，MySQL 会自动执行 `COMMIT`。但如果该语句返回错误，则根据[错误情况](https://dev.mysql.com/doc/refman/5.7/en/innodb-error-handling.html)执行 `COMMIT` 或 `ROLLBACK`。

如何修改当前会话的提交模式？

```sql
SHOW VARIABLES LIKE 'AUTOCOMMIT';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+

--1 或者 ON 表示启用， 0 或者 OFF 表示禁用
SET AUTOCOMMIT = 0;
```

注意：

* 关闭后，会话将始终开启一个事务。直到你显式提交或回滚该事务后，一个新事务又被开启。
* 如果一个关闭了 `autocommit` 的会话没有显式提交事务，然后会话被关闭，MySQL 将回滚该事务。
* 有一些命令，在执行之后会强制执行 `COMMIT` 提交当前的活动事务。例如：
  - `ALTER TABLE`
  - `LOCK TABLES`

# 提交多语句事务

如何在一个事务中组合多条 SQL 语句（multiple-statement transaction）？有两种方式：

1. 方式一：显式关闭当前会话的 `autocommit`，然后提交或回滚事务。

   ```sql
   SET autocommit=0;
   INSERT INTO parent VALUES (10, 'Heikki');
   COMMIT;
   ```

2. 方式二：如果不想关闭 `autocommit`，可以通过 [`START TRANSACTION`](https://dev.mysql.com/doc/refman/5.7/en/commit.html) 或 [`BEGIN`](https://dev.mysql.com/doc/refman/5.7/en/commit.html) 语句显式开启事务，然后通过 [`COMMIT`](https://dev.mysql.com/doc/refman/5.7/en/commit.html)或 [`ROLLBACK`](https://dev.mysql.com/doc/refman/5.7/en/commit.html) 语句显式结束事务。

   ```sql
   START TRANSACTION;
   INSERT INTO parent VALUES (15, 'John');
   INSERT INTO parent VALUES (20, 'Paul');
   DELETE FROM parent WHERE b = 'Heikki';
   ROLLBACK;
   ```

最终结果：

```sql
SELECT * FROM parent;
+------+--------+
|  id  | name   |
+------+--------+
|  10  | Heikki |
+------+--------+
```

# 在事务中混合使用存储引擎问题

MySQL 服务器层不管理事务，事务是由下层的存储引擎实现的。所以在同一个事务中，使用多种存储引擎是不可靠的。

如果在事务中混合使用了事务型和非事务型的表（例如 `InnoDB` 和 `MyISAM` 表），可能会有意想不到的情况发生。请看下例：

```sql
--引入一张 MyISAM 表
CREATE TABLE `people` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `last_name` varchar(50) NOT NULL,
  `first_name` varchar(50) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM;
```

示例一，在事务中混合使用存储引擎，出现报错：

```sql
START TRANSACTION;
--执行成功
INSERT INTO parent(name) VALUES('Heikki');
--执行失败
INSERT INTO people(last_name, first_name) VALUES('pete', 'Lee');
1785 - When @@GLOBAL.ENFORCE_GTID_CONSISTENCY = 1, updates to non-transactional tables can only be done in either autocommitted statements or single-statement transactions, and never in the same statement as updates to transactional tables.
```

示例二，当事务回滚，非事务型的表上的变更无法撤销，这会导致数据库处于不一致的状态，这种情况很难修复，事务的最终结果将无法确定。

```sql
START TRANSACTION;
INSERT INTO people(last_name, first_name) VALUES('pete', 'Lee');
INSERT INTO parent(name) VALUES('Heikki');
--parent表（InnoDB）回滚成功，people表（MyISAM）回滚失败
ROLLBACK;
```

所以，为每张表选择合适的存储引擎非常重要。

# 参考

《高性能 MySQL》

https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html

https://dev.mysql.com/doc/refman/5.7/en/sql-syntax-transactions.html