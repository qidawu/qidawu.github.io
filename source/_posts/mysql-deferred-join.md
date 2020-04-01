---
title: MySQL 延迟关联优化超多分页场景
date: 2019-11-26 15:39:05
updated:
tags: MySQL
typora-root-url: ..
---

# 一个例子

场景：

* 订单表数据量：3000 万。
* 查询最近 7 天的订单，并做分页、分片。

表结构：

```sql
CREATE TABLE `t_order` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `order_no` varchar(50) NOT NULL,
  ...
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_order_no` (`order_no`) USING BTREE,
  KEY `idx_create_time` (`create_time`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

超多分页场景如下：

```sql
explain select * from t_order 
where create_time between '2019-10-17' and '2019-10-25' 
limit 1000000, 10;

+----+-------------+---------+-------+-----------------+-----------------+---------+------+---------+-----------------------+
| id | select_type | table   | type  | possible_keys   | key             | key_len | ref  | rows    | Extra                 |
+----+-------------+---------+-------+-----------------+-----------------+---------+------+---------+-----------------------+
|  1 | SIMPLE      | t_order | range | idx_create_time | idx_create_time | 5       | NULL | 3775048 | Using index condition |
+----+-------------+---------+-------+-----------------+-----------------+---------+------+---------+-----------------------+
```

虽然走了 `idx_create_time` 索引，但仍然是个慢查询，扫描行数过多。

# 超多分页的问题是什么？

> 随着偏移量 `offset` 的增加，MySQL 需要花费大量的时间来扫描需要丢弃的数据。本质上就是 `offset` 过大导致的大量回表 I/O 查询。

如果能减少这种大量的回表查询，就能提升查询性能。

# 概念介绍

## 什么是延迟关联优化？

什么是“延迟关联”？

>  通过使用覆盖索引查询返回需要的主键，再根据主键关联原表获得需要的数据。 

参考两个材料：

《高性能 MySQL》P194：

![deferred_join_1](/img/mysql/deferred_join_1.png)

《阿里巴巴 Java 开发手册》：

![deferred_join_2](/img/mysql/deferred_join_2.png)

## 什么是覆盖索引？

查询的列被所建的辅助索引所覆盖，无需回表：

```sql
explain select id from t_order 
where create_time between '2019-10-17' and '2019-10-25' 
limit 1000000, 10;

+----+-------------+---------+-------+-----------------+-----------------+---------+------+---------+--------------------------+
| id | select_type | table   | type  | possible_keys   | key             | key_len | ref  | rows    | Extra                    |
+----+-------------+---------+-------+-----------------+-----------------+---------+------+---------+--------------------------+
|  1 | SIMPLE      | t_order | range | idx_create_time | idx_create_time | 5       | NULL | 3775048 | Using where; Using index |
+----+-------------+---------+-------+-----------------+-----------------+---------+------+---------+--------------------------+
```

上述查询字段改为 `id` 后，执行计划中新增： `Extra=Using index`，表示走覆盖索引，无需回表，查询速度快了 N 倍。

# 延迟关联优化

延迟关联优化涉及到了 SQL 优化的两个重要概念：覆盖索引和回表。

* 通过覆盖索引在辅助索引上完成所有扫描、过滤、排序（利用索引有序）和分页；
* 最后通过主键回表查询，最大限度减少回表查询的 I/O 次数。

SQL 改造如下：

```sql
explain select * from t_order t 
inner join (
    select id from t_order 
    where create_time between '2019-10-17' and '2019-10-25' 
    limit 1000000, 10
) e 
on t.id = e.id;

+----+-------------+---------------+--------+-----------------+-----------------+---------+------+---------+--------------------------+
| id | select_type | table         | type   | possible_keys   | key             | key_len | ref  | rows    | Extra                    |
+----+-------------+---------------+--------+-----------------+-----------------+---------+------+---------+--------------------------+
|  1 | PRIMARY     | <derived2>    | ALL    | NULL            | NULL            | NULL    | NULL | 1000010 | NULL                     |
|  1 | PRIMARY     | t             | eq_ref | PRIMARY         | PRIMARY         | 8       | e.id |       1 | NULL                     |
|  2 | DERIVED     | t_order       | range  | idx_create_time | idx_create_time | 5       | NULL | 3775048 | Using where; Using index |
+----+-------------+---------------+--------+-----------------+-----------------+---------+------+---------+--------------------------+
```

优化前：

1. 辅助索引查询，得到 id
2. id 逐一回表查询**（1000000  + 10 次回表）**
3. 查询结果放弃前 offset 行，返回 limit 行

优化后：

1. 辅助查询覆盖查询，得到 id
2. 查询结果放弃前 offset 行，返回 limit 行
3. **只需 limit 条 id 回表查询**，大大减少回表查询的 I/O 次数

# 参考

https://dev.mysql.com/doc/refman/5.7/en/derived-tables.html

http://mysql.taobao.org/monthly/2017/03/05/