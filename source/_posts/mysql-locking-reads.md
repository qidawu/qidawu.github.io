---
title: MySQL 加锁读机制总结
date: 2018-10-21 21:39:16
updated:
tags: MySQL
---

举个例子，有一张 parent 和 child 表：

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

`InnoDB` 支持两种类型的 [加锁读（Locking Reads）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_locking_read)，为事务操作提供额外的**安全性**：

* `SELECT ... LOCK IN SHARE MODE ` or `SELECT ... FOR SHARE` in MySQL 8.0.1，在检索行上设置共享锁（s-lock）
  * 其它事务允许读取检索行，但不允许更新或删除，一直阻塞等待，直到该事务结束。
* `SELECT ... FOR UPDATE` 在检索行上设置排它锁（x-lock）
  * 其它事务不允许更新或删除
  * 不允许加共享锁读取 `SELECT ... LOCK IN SHARE MODE`
  * 如果事务隔离级别为 `SERIALIZABLE`，不允许读取（因为该级别的读取默认需要获得共享读锁）
  * 上述操作将一直阻塞等待，直到该事务结束。

共享锁和排它锁的详情请参考：《[MySQL 并发控制机制总结](/2018/10/20/mysql-concurrency-control/)》。

# LOCK IN SHARE MODE

举个例子：

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

# FOR UPDATE

举个例子：

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

因此，解决办法是通过 `SELECT ... LOCK IN SHARE MODE ` 设置共享锁：

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
* 锁定读取有可能产生**死锁**，具体取决于事务的隔离级别。

# 参考

《高性能 MySQL》

https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html
