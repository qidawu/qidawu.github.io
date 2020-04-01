---
title: MySQL 锁粒度与索引总结
date: 2019-02-22 22:43:57
updated:
tags: MySQL
typora-root-url: ..
---

# 锁的粒度

所谓的锁粒度，就是在**锁的开销**和**数据的安全性**之间寻求平衡，这种平衡当然也会影响到性能：

> 一种提高共享资源并发性的方式就是让锁定对象更有**选择性**。尽量只锁定需要修改的部分数据，而不是所有资源。更理想的方式是，只对会修改的数据片进行精确的锁定。任何时候，在给定的资源上，锁定的数据量越少，则系统的并发程度越高，只要相互之间不发生冲突即可。
>
> 问题是加锁也需要消耗资源。锁的各种操作，包括获得锁、检查锁是否已经解除、释放锁等，都会增加系统的开销。如果系统花费大量的时间来管理锁，而不是存取数据，那么系统的性能可能会因此受到影响。

`InnoDB` 存储引擎目前有以下两种锁粒度：

## 表锁

表锁（Table Lock）是 MySQL 中最基本的锁粒度，并且是开销最小的粒度。

## 行锁

行锁（Row Lock）可以最大程度的支持并发处理，同时也带来了最大的锁开销。行锁只在存储引擎层实现，而不在 MySQL 服务器层。`InnoDB` 的锁粒度就是到行级别的。

# 锁粒度与索引

以一个例子总结锁粒度与索引的关系：

```sql
CREATE TABLE `child` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `parent_id` bigint(20) NOT NULL,
  `name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_parent_id` (`parent_id`) USING BTREE
) ENGINE=InnoDB;
```

1、`InnoDB` 行锁是通过给索引上的索引项加锁来实现的，只有通过索引条件检索数据，`InnoDB` 才使用行锁，否则，`InnoDB` 将使用表锁：

```sql
--开启事务 T1
START TRANSACTION;

--查看执行计划，全表扫描（type=ALL）
EXPLAIN SELECT * FROM child WHERE name = 'D' FOR UPDATE;
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | child | ALL  | NULL          | NULL | NULL    | NULL |    5 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+

--执行查询，加表锁
SELECT * FROM child WHERE name = 'D' FOR UPDATE;
+----+-----------+------+
| id | parent_id | name |
+----+-----------+------+
|  4 |         3 | D    |
+----+-----------+------+

--开启事务 T2
START TRANSACTION;

--查看执行计划，命中索引 idx_parent_id	
EXPLAIN SELECT * FROM child WHERE parent_id = 4 FOR UPDATE;
+----+-------------+-------+------+---------------+---------------+---------+-------+------+-------+
| id | select_type | table | type | possible_keys | key           | key_len | ref   | rows | Extra |
+----+-------------+-------+------+---------------+---------------+---------+-------+------+-------+
|  1 | SIMPLE      | child | ref  | idx_parent_id | idx_parent_id | 8       | const |    1 | NULL  |
+----+-------------+-------+------+---------------+---------------+---------+-------+------+-------+

--执行查询，由于事务 T1 加了表锁，事务 T2 对 parent_id = 4 索引项的行锁被阻塞，一直等待
SELECT * FROM child WHERE parent_id = 4 FOR UPDATE;
```

2、由于 MySQL 的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现锁冲突的。应用设计的时候要注意这一点：

```sql
--开启事务 T1
START TRANSACTION;

--查看执行计划，命中索引 idx_parent_id
EXPLAIN SELECT * FROM child WHERE parent_id = 2 AND name = 'A' FOR UPDATE;
+----+-------------+-------+------+---------------+---------------+---------+-------+------+-------------+
| id | select_type | table | type | possible_keys | key           | key_len | ref   | rows | Extra       |
+----+-------------+-------+------+---------------+---------------+---------+-------+------+-------------+
|  1 | SIMPLE      | child | ref  | idx_parent_id | idx_parent_id | 8       | const |    2 | Using where |
+----+-------------+-------+------+---------------+---------------+---------+-------+------+-------------+

--执行查询
SELECT * FROM child WHERE parent_id = 2 AND name = 'A' FOR UPDATE;
+----+-----------+------+
| id | parent_id | name |
+----+-----------+------+
|  1 |         2 | A    |
+----+-----------+------+

--开启事务 T2
START TRANSACTION;

--查看执行计划，命中索引 idx_parent_id
EXPLAIN SELECT * FROM child WHERE parent_id = 2 AND name = 'C' FOR UPDATE;
+----+-------------+-------+------+---------------+---------------+---------+-------+------+-------------+
| id | select_type | table | type | possible_keys | key           | key_len | ref   | rows | Extra       |
+----+-------------+-------+------+---------------+---------------+---------+-------+------+-------------+
|  1 | SIMPLE      | child | ref  | idx_parent_id | idx_parent_id | 8       | const |    2 | Using where |
+----+-------------+-------+------+---------------+---------------+---------+-------+------+-------------+

-- 执行查询，虽然 T1、T2 访问不同行的记录，但由于使用了相同的索引键 parent_id = 2，出现锁冲突，从而阻塞，一直等待
SELECT * FROM child WHERE parent_id = 2 AND name = 'C' FOR UPDATE;
```

3、当表有多个索引的时候，不同的事务可以使用不同的索引锁定不同的行，另外，不论是使用主键索引、唯一索引或普通索引，`InnoDB` 都会使用行锁来对数据加锁。

4、即便在条件中使用了索引字段，但是否使用索引来检索数据是由 MySQL 通过判断不同执行计划的代价来决定的，如果 MySQL 认为全表扫描效率更高，比如对一些很小的表，它就不会使用索引，这种情况下 `InnoDB` 将使用表锁，而不是行锁。因此，在分析锁冲突时，别忘了检查 SQL 的执行计划，以确认是否真正使用了索引：

```sql
--开启事务 T1
START TRANSACTION;

--查看执行计划，虽然使用了索引 idx_parent_id，但 MySQL 认为全表扫描效率更高，因此实际上没有使用索引
EXPLAIN SELECT * FROM child WHERE parent_id = 2 FOR UPDATE;
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | child | ALL  | idx_parent_id | NULL | NULL    | NULL |    5 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+

--虽然使用了索引 idx_parent_id，但由于进行了全表扫描，因此实际使用表锁
SELECT * FROM child WHERE parent_id = 2 FOR UPDATE;
+----+-----------+------+
| id | parent_id | name |
+----+-----------+------+
|  1 |         2 | A    |
|  2 |         2 | C    |
|  3 |         2 | C    |
+----+-----------+------+

--开启事务 T2
START TRANSACTION;

--执行查询，由于事务 T1 加了表锁，事务 T2 对 parent_id = 4 索引项的行锁被阻塞，一直等待
SELECT * FROM child WHERE parent_id = 4 FOR UPDATE;
```

# 参考

《高性能 MySQL》