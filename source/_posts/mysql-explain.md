---
title: MySQL 执行计划总结
date: 2017-12-21 22:23:06
updated:
tags: MySQL
typora-root-url: ..
---

`EXPLAIN` 语句提供有关 MySQL 优化器如何执行语句的信息。能够用于 `SELECT`、`DELETE`、`INSERT`、`REPLACE`、`UPDATE` 语句。

`EXPLAIN` 为 `SELECT` 语句中使用的每个表返回一行信息 。它按照 MySQL 在处理语句时读取它们的顺序列出了输出中的表。MySQL 使用嵌套循环连接方法解析所有表连接（MySQL resolves all joins using a nested-loop join method）。这意味着 MySQL 从第一个表中读取一行，然后在第二个表、第三个表中找到匹配的行，以此类推。处理完所有表后，MySQL 输出所选的列，并在表列表中回溯，直到找到一个有更多匹配行的表。从该表中读取下一行，然后继续下一个表。

`EXPLAIN` 输出列如下：

| Column                                                       | JSON Name              | Meaning                                        |
| ------------------------------------------------------------ | ---------------------- | ---------------------------------------------- |
| [`id`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_id) | `SELECT` 标识符        | The `SELECT` identifier                        |
| [`select_type`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_select_type) | `SELECT` 类型          | The `SELECT` type                              |
| [`table`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_table) | 引用的表名             | The table for the output row                   |
| [`partitions`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_partitions) | 匹配的分区             | The matching partitions                        |
| [`type`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_type) | 连接类型               | The join type                                  |
| [`possible_keys`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_possible_keys) | 可选的索引             | The possible indexes to choose                 |
| [`key`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_key) | 实际选择的索引         | The index actually chosen                      |
| [`key_len`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_key_len) | 实际所选 key 的长度    | The length of the chosen key                   |
| [`ref`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_ref) | 与索引比较的列         | The columns compared to the index              |
| [`rows`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_rows) | 扫描行数               | Estimate of rows to be examined                |
| [`filtered`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_filtered) | 按表条件过滤的行百分比 | Percentage of rows filtered by table condition |
| [`Extra`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_extra) | 附加信息               | Additional information                         |

# id

`id` 列的编号是 `SELECT` 的序列号，有几个 `SELECT` 就有几个 `id`。`id` 值越大执行优先级越高，`id` 值相同则从上往下执行，`id` 值为 `NULL` 则最后执行。

# select_type

表示查询类型是简单查询还是复杂查询。常见 `SELECT` 类型如下：

| `select_type` Value                                          | Meaning                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `SIMPLE`                                                     | Simple [`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html) (not using [`UNION`](https://dev.mysql.com/doc/refman/5.7/en/union.html) or subqueries) |
| `PRIMARY`                                                    | Outermost [`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html) |
| [`UNION`](https://dev.mysql.com/doc/refman/5.7/en/union.html) | Second or later [`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html) statement in a [`UNION`](https://dev.mysql.com/doc/refman/5.7/en/union.html) |
| `UNION RESULT`                                               | Result of a [`UNION`](https://dev.mysql.com/doc/refman/5.7/en/union.html) |
| [`SUBQUERY`](https://dev.mysql.com/doc/refman/5.7/en/optimizer-hints.html#optimizer-hints-subquery) | First [`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html) in subquery |
| `DERIVED`                                                    | Derived table                                                |
| `MATERIALIZED`                                               | Materialized subquery                                        |

# table

表示输出行所引用的表名，特殊情况如下：

- <union*M*,*N*>：该行指的是 `id` 值为 *M* 和 *N* 的并集。当有 `UNION` 时，`UNION RESULT` 的 `table` 列的值为 <union*M*,*N*>，表示参与并集的 `id` 查询编号为 *M* 和 *N*。
- <derived*N*>：当 `FROM` 子句中有子查询时， `table` 列为 <derived*N*>，表示当前查询依赖于 id=N 的查询结果，于是先执行 id=N 的查询。
- <subquery*N*>

# type

单表查询的性能对比：`const` > `ref` > `range` > `index` > `ALL`。一般来说，得保证查询达到 `range` 级别，最好达到 `ref` 级别。

## system

该表只有一行。是 `const` 连接类型的特例。

## const

该表最多只有一个匹配行，该行在查询开始时读取。因为只有一行，所以优化器的其余部分可以将这一行中列的值视为常量。`const` 表非常快，因为它们只能读取一次。

当主键索引（`PRIMARY KEY` ）或唯一索引（`UNIQUE KEY`）与常量值比较时使用 `const` 类型。如下：

```sql
SELECT * FROM tbl_name WHERE primary_key = 1;

SELECT * FROM tbl_name WHERE primary_key_part1 = 1 AND primary_key_part2 = 2;

SELECT * FROM tbl_name WHERE unique_key = '001';

SELECT * FROM tbl_name WHERE unique_key_part1 = '001' AND unique_key_part2 = '002';

SELECT * FROM ref_table,other_table
  WHERE ref_table.unique_key_column = other_table.unique_key_column
  AND other_table.unique_key_column = '001';
```

## eq_ref

对于 `other_table` 中的每行，仅从 `ref_table` 中读取唯一一行。`eq_ref` 类型用于主键索引（`PRIMARY KEY` ）或 `NOT NULL` 的唯一索引（`UNIQUE KEY`），且索引被表连接所使用时。除了 `system` 和 `const` 类型之外，这是最好的连接类型。`select_type=SIMPLE` 简单查询类型不会出现这种类型。

例子：

```sql
SELECT * FROM ref_table,other_table
  WHERE ref_table.unique_key_column = other_table.column;
  
SELECT * FROM ref_table,other_table
  WHERE ref_table.unique_key_column_part1 = other_table.column
  AND ref_table.unique_key_column_part2 = 1;
```

## ref

对于 `other_table` 中的每行，从 `ref_table` 中读取所有匹配行。`ref` 类型用于普通索引或联合索引的最左前缀列（`leftmost prefix of the key`），即无法根据键值查询到唯一一行。如果使用的索引仅匹配几行结果，则也是一种很好的连接类型。

例子：

```sql
SELECT * FROM ref_table WHERE key_column = expr;

SELECT * FROM ref_table WHERE key_column_part1 = expr;

SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column = other_table.column;
  
SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column_part1 = other_table.column
  AND ref_table.key_column_part2 = 1;
```

## range

使用索引进行范围查询时，例如：`=`, `<>`, `>`, `>=`, `<`, `<=`,  `<=>`, `IS NULL`, `BETWEEN`, `LIKE`, `IN()`。

```sql
SELECT * FROM tbl_name
  WHERE key_column = 10;

SELECT * FROM tbl_name
  WHERE key_column BETWEEN 10 and 20;

SELECT * FROM tbl_name
  WHERE key_column IN (10,20,30);

SELECT * FROM tbl_name
  WHERE key_part1 = 10 AND key_part2 IN (10,20,30);
```

## index

索引扫描，类似于 `ALL` 全表扫描。以下情况发生：

* 覆盖索引（`covering index`）。此时 `Extra` 列显示 `Using index`。覆盖索引扫描通常比全表扫描速度更快，因为其存储空间更小。例子：

  ```sql
  SELECT primary_key FROM tbl_name;
  
  SELECT unique_key FROM tbl_name;
  ```

## ALL

全表扫描。此时必须增加索引优化查询。

# possible_keys

表示 MySQL 可选的索引。

如果此列为 `NULL`，表示 MySQL 没有可选的索引。此时，可以检查 `WHERE` 子句是否引用了某些适合建立索引的列，建立索引以提升查询性能。

# key

表示 MySQL 实际选择的索引。

* 如果此列为 `NULL`，表示 MySQL 没有找到可用于提高查询性能的索引。
* 如果 `possible_keys NOT NULL`，但 `key NULL`，可能是因为表中数据不多，MySQL 认为索引对此查询帮助不大，选择了全表扫描。

如需强制 MySQL 使用或忽略 `possible_keys` 中列出的索引，可以在查询中使用 `FORCE INDEX`、`USE INDEX` 或 `IGNORE INDEX`。详见：[索引提示](https://dev.mysql.com/doc/refman/5.7/en/index-hints.html)。

# key_len

表示 MySQL 实际选择的索引长度。如果该索引为联合索引，可用于判断 MySQL 实际使用了联合索引中的多少个字段。如果 `key` 列为 `NULL`，`key_len` 列也为 `NULL`。

`key_len` 计算规则如下：

* 使用 `NULL` 需要额外增加 1 Byte 记录是否为 `NULL`。并且进行比较和计算时要对 `NULL` 值做特别的处理，因此尽可能把所有列定义为 `NOT NULL`。
* 各个类型：
  * 整数类型
    - `TINYINT` 1 Byte
    - `SMALLINT` 2 Bytes
    - `MEDIUMINT` 3 Bytes
    - `INT` 4 Bytes
    - `BIGINT` 8 Bytes
  * 日期与时间类型
    - `DATE` 3 Bytes
    - `TIMESTAMP` 4 Bytes
    - `DATETIME` 8 Bytes
  * 字符串类型
    - `char(n)`：如果字符集为 `utf8`，则长度为 3n Bytes
    - `varchar(n)`：如果字符集为 `utf8`，则长度为 3n + 2 Bytes。额外 2 Bytes 用于存储长度。
* 各个字符集：
  * `latin1` 编码一个字符 1 Byte
  * `gbk` 编码一个字符 2 Bytes
  * `utf8` 编码一个字符 3 Bytes
  * `utf8mb4` 编码一个字符 4 Bytes
* 创建索引的时候可以指定索引的长度，例如：`alter table test add index idx_username (username(30));`。长度 30 指的是字符的个数。

* 索引最大长度为 768 Bytes，也就是说字符集为 `utf8` 的情况下，`n` 最大只能为 (768 - 2 (存储长度) - 1 (是否为 `NULL`)) / 3 = 765 / 3 = 255 个字符。因此当字符串过长时，MySQL 最多会将开头 255 个字符串截取出来作为索引。一个例子：

```sql
CREATE TABLE `student` (
  `id` int(11) NOT NULL,
  `username` varchar(256) DEFAULT NULL,
  `password` varchar(1) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_username` (`username`(255)) USING BTREE,
  KEY `idx_password` (`password`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- key_len: 255 * 3 + 2 + 1 = 768 Bytes
mysql> explain select username from student where username = 'pete';
+----+-------------+---------+------+---------------+--------------+---------+-------+------+-------------+
| id | select_type | table   | type | possible_keys | key          | key_len | ref   | rows | Extra       |
+----+-------------+---------+------+---------------+--------------+---------+-------+------+-------------+
|  1 | SIMPLE      | student | ref  | idx_username  | idx_username | 768     | const |    1 | Using where |
+----+-------------+---------+------+---------------+--------------+---------+-------+------+-------------+

-- key_len: 1 * 3 + 2 + 1 = 6 Bytes
mysql> explain select password from student where password = 'pete';
+----+-------------+---------+------+---------------+--------------+---------+-------+------+--------------------------+
| id | select_type | table   | type | possible_keys | key          | key_len | ref   | rows | Extra                    |
+----+-------------+---------+------+---------------+--------------+---------+-------+------+--------------------------+
|  1 | SIMPLE      | student | ref  | idx_password  | idx_password | 6       | const |    1 | Using where; Using index |
+----+-------------+---------+------+---------------+--------------+---------+-------+------+--------------------------+
```

# ref

`ref` 显示与 `key` 列（实际选择的索引）比较的内容，可选值：

* 列名
* `const`：常量值
* `func`：值为某些函数的结果
* `NULL`：范围查询（`type=range`）

# rows

表示 MySQL 认为执行查询必须扫描的行数。

对于 InnoDB 表，此数字是估计值，可能并不总是准确。

# Extra

这一列显示的是额外信息。如果想要查询越快越好，需特别留意 `Extra` 列是否出现 `Using filesort` 或 `Using temporary`。

常见的重要值如下：

## Using index

使用了覆盖索引。

## Using where

使用 `WHERE` 条件过滤结果，但查询的列未被索引覆盖。

## Using index condition

查询的列不完全被索引覆盖，`WHERE` 条件中是一个前导列的范围。

## Using temporary

MySQL 需要创建一张临时表来处理查询。通常发生于查询包含 `DISTINCT`、`GROUP BY` 或 `ORDER BY` 子句等需要数据去重的场景。出现这种情况一般是要进行优化的，首先想到的是用索引进行优化。

## Using filesort

将用外部排序而不是索引排序，数据较少时从内存排序，否则需要在磁盘完成排序。这种情况下一般考虑使用索引进行优化。

详见：[Section 8.2.1.14, “ORDER BY Optimization”](https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html)

# 参考

https://dev.mysql.com/doc/refman/5.7/en/execution-plan-information.html

https://dev.mysql.com/doc/refman/5.7/en/analyze-table.html

*MySQL Workbench* has a Visual Explain capability that provides a visual representation of [EXPLAIN](https://dev.mysql.com/doc/refman/5.7/en/explain.html) output. See [Tutorial: Using Explain to Improve Query Performance](https://dev.mysql.com/doc/workbench/en/wb-tutorial-visual-explain-dbt3.html).