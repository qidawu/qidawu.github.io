---
title: MySQL 索引优化
date: 2017-11-30 18:55:53
updated:
tags: MySQL
---

索引优化应该是对查询性能优化最有效的手段了。索引能够轻易将查询性能提高几个数量级，“最优”的索引有时比一个“好的”索引性能要好两个数量级。

索引是存储引擎用于快速找到记录的一种数据结构。在 MySQL 中，索引是在存储引擎层而不是服务器层实现的。所以，并没有统一的索引标准：不同存储引擎的索引的工作方式并不一样，也不是所有的存储引擎都支持所有类型的索引。即使多个存储引擎支持同一种类型的索引，其底层的实现也可能不同。

MySQL 支持的常见索引类型：

*   B-Tree 索引
*   Hash index（哈希索引）
*   R-Tree index（空间数据索引）
*   Full-text index（全文索引）
*   ……

本文只探讨最常用的 B-Tree 索引。

# B-Tree 索引

当人们谈论索引的时候，如果没有特别指明类型，那多半说的是 B-Tree 索引，它使用 B-Tree 数据结构来存储数据。（存储引擎 InnoDB 使用的是 B+Tree）
索引可以包含一个或多个列的值。如果索引包含多个列，那么列的顺序至关重要，因为 MySQL 只能高效地使用索引的 **最左前缀列**。创建一个包含两个列的索引，和创建两个只包含一列的索引是大不相同的，下面看个例子：

```mysql
CREATE TABLE People (
   last_name  varchar(50)    not null,
   first_name varchar(50)    not null,
   dob        date           not null,
   gender     enum('m', 'f') not null,
   key(last_name, first_name, dob)
);
```

可以使用该 B-Tree 索引的查询类型：

## 全值匹配

全值匹配（Match the full value）：

```mysql
SELECT * 
FROM people
WHERE last_name = 'Wu' AND first_name = 'Qida' AND dob = '2018-01-01';
```

## 匹配最左前缀

匹配最左前缀（Match a leftmost prefix）：

```mysql
SELECT * 
FROM people
WHERE last_name = 'Wu';

SELECT * 
FROM people
WHERE last_name = 'Wu' AND first_name = 'Qida';
```

## 匹配列前缀

匹配列前缀（Match a column prefix）：

```mysql
SELECT * 
FROM people
WHERE last_name LIKE 'W%';
```

## 匹配范围值

匹配范围值（Match a range of values）：

```mysql
SELECT * 
FROM people
WHERE last_name BETWEEN 'Wu' AND 'Li';
```

## 精确匹配某一列，并范围匹配另外一列

精确匹配某一列，并范围匹配另外一列（Match one part exactly and match a range on another part）：

```mysql
SELECT * 
FROM people
WHERE last_name = 'Wu' And first_name LIKE 'Q%';
```
## 只访问索引的查询

只访问索引的查询（Index-only queries）：

# 高性能的索引策略

## 索引选择性（Index Selectivity）

索引的选择性是指，不重复的索引值（也称为基数，cardinality）和数据表的记录总数（#T）的比值，范围从 `1/#T` 到 `1` 之间，

索引的选择性越高则查询效率越高，因为选择性高的索引可以让 MySQL 在查找时过滤掉更多的行。唯一索引的选择性是 1，这是最好的索引选择性，性能也是最好的。

下面显示如何计算某列的**平均完整性**：

```mysql
SELECT COUNT(DISTINCT last_name) / COUNT(*) AS Selectivity
FROM people;

Selectivity: 0.0312
```
只看平均选择性有时是不够的，需考虑最坏或特殊情况下的选择性（即值的分布，是否有某些值占比过多？）。

## 选择合适的索引列顺序



## 使用独立的列

“独立的列”是指索引列不能是表达式的一部分，也不能是函数的参数。

例如，下面这个查询无法使用 actor_id 列的索引：

```mysql
SELECT * 
FROM people
WHERE id + 1 = 5;
```

凭肉眼很容易看出 `WHERE` 中的表达式其实等价于 `id  = 4` ，但是 MySQL 无法自动解析这个方程式。这完全是用户行为。我们应该养成简化 `WHERE` 条件的习惯，始终将索引列单独放在比较符号的一侧。

下面是另一个常见的错误：

```mysql
SELECT ... 
WHERE TO_DAYS(CURRENT_DATE) - TO_DAYS(date_col) <= 10;
```

# 三星索引



# 相关命令

下面介绍一些查看索引的常用命令：

## DESC

`DESC` 命令查看表结构时，可以看到索引列 `Key` ，共有三种类型：`PRI`、`UNI`、`MUL`

```
$ DESC table_name;

+---------+--------------+------+-----+---------+-------+
| Field   | Type         | Null | Key | Default | Extra |
+---------+--------------+------+-----+---------+-------+
| FID     | int(11)      | NO   | PRI | NULL    |       |
| FKEY    | varchar(50)  | NO   | UNI | NULL    |       |
| FVALUE  | varchar(500) | YES  | MUL | NULL    |       |
| FDESC   | varchar(50)  | YES  | MUL | NULL    |       |
| FCACHED | int(1)       | NO   |     | 1       |       |
+---------+--------------+------+-----+---------+-------+
```

* `PRI` 表示主键索引（PRIMARY KEY）。
* `UNI` 表示唯一性索引（UNIQUE KEY），值不能重复。
* `MUL` 表示非唯一性索引（MULTIPLE KEY） ，值可重复。

## SHOW INDEX

`SHOW INDEX` 可以以列为单位，查看该表索引的具体信息，例如：

* 索引名 `Key_name`
* 索引唯一性 `Non_unique`
* 联合索引中的顺序 `Seq_in_index`
* 列名 `Column_name`
* 索引类型 `Index_type`

```
$ SHOW INDEX FROM table_name;

+------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
|   Table    | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| table_name |          0 |  PRIMARY |            1 | FID         | A         |         105 |     NULL | NULL   |      | BTREE      |         |               |
| table_name |          0 |  UK_FKEY |            1 | FKEY        | A         |         105 |     NULL | NULL   |      | BTREE      |         |               |
| table_name |          1 |  IDX_V_D |            1 | FVALUE      | A         |         105 |     NULL | NULL   |      | BTREE      |         |               |
| table_name |          1 |  IDX_V_D |            2 | FDESC       | A         |         105 |     NULL | NULL   |      | BTREE      |         |               |
+------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
```

##  SHOW CREATE TABLE

`SHOW CREATE TABLE` 可以查看该表的建表语句，留意最后三行：

```
SHOW CREATE TABLE table_name;

CREATE TABLE `table_name` (
  `FID` int(11) NOT NULL,
  `FKEY` varchar(50) NOT NULL,
  `FVALUE` varchar(200) NOT NULL,
  `FDESC` varchar(50) DEFAULT NULL,
  `FCACHED` int(1) NOT NULL DEFAULT '1',
  PRIMARY KEY (`FID`),
  UNIQUE KEY `UK_FKEY` (`FKEY`),
  KEY `IDX_V_D` (`FVALUE`, `FDESC`)
)
```

# 参考

《[SHOW COLUMNS Syntax](https://dev.mysql.com/doc/refman/5.7/en/show-columns.html)》

《[MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)》