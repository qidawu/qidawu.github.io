---
title: MySQL 执行计划总结
date: 2019-10-01 22:23:06
updated:
tags: MySQL
typora-root-url: ..
---

`EXPLAIN` 语句提供有关 MySQL 优化器如何执行语句的信息。能够用于 `SELECT`、`DELETE`、`INSERT`、`REPLACE`、`UPDATE` 语句。

`EXPLAIN` 为 `SELECT` 语句中使用到的每张表输出一行信息 。它按照 MySQL 在处理 `SELECT` 语句时的读取顺序来列出各张表。

MySQL 使用嵌套循环连接算法（NLJ）来解析所有的表连接（MySQL resolves all joins using a nested-loop join method）。详见另一篇。

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
  
  SELECT COUNT(primary_key) FROM tbl_name;
  
  SELECT COUNT(unique_key) FROM tbl_name;
  ```

## ALL

[全表扫描](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_full_table_scan)。此时必须增加索引优化查询。

全表扫描发生的情况如下：

* 小表，此时全表扫描比二级索引扫描再回表的速度要快；
* `ON` 或 `WHERE` 子句没有可用的索引；
* 查询的字段虽然使用了索引，但查询条件覆盖的范围太大以至于还不如全表扫描。优化方式详见：[Section 8.2.1.1, “WHERE Clause Optimization”](https://dev.mysql.com/doc/refman/5.7/en/where-optimization.html)
* 使用了区分度（cardinality）低的索引，索引扫描范围太大以至于还不如全表扫描。如果是统计不准，可以用 `ANALYZE TABLE` 语句优化：[Section 13.7.2.1, “ANALYZE TABLE Syntax”](https://dev.mysql.com/doc/refman/5.7/en/analyze-table.html)

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

* `InnoDB` 索引最大长度为 767 Bytes，引自[官方文档](https://dev.mysql.com/doc/refman/5.7/en/create-table.html)：

  > *key_part*:
  > *col_name* [(*length*)] [ASC | DESC]
  >
  > *index_type*:
  > USING {BTREE | HASH}
  >
  > > Prefixes, defined by the *length* attribute, can be up to 767 bytes long for `InnoDB` tables or 3072 bytes if the [`innodb_large_prefix`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_large_prefix) option is enabled. For `MyISAM` tables, the prefix length limit is 1000 bytes.

举个例子，在字符集为 `utf8` 的情况下，`n` 最大只能为 `(767 - 2 (存储长度)) / 3 = 765 / 3 = 255 个字符`。因此当字符串过长时，MySQL 最多会将开头 255 个字符串截取出来作为索引。一个例子：

```sql
CREATE TABLE `student` (
  `id` int(11) NOT NULL,
  `username` varchar(256) DEFAULT NULL,
  `password` varchar(1) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_username` (`username`(255)) USING BTREE,
  KEY `idx_password` (`password`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- key_len: 255 * 3 + 2 + 1 = 768 Bytes (额外增加 1 Byte 记录是否为 NULL)
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

如果使用过长的索引，例如修改了字符串编码类型、增加联合索引列，则报错如下：

```
[Err] 1071 - Specified key was too long; max key length is 767 bytes
```

# ref

`ref` 显示与 `key` 列（实际选择的索引）比较的内容，可选值：

* 列名
* `const`：常量值
* `func`：值为某些函数的结果
* `NULL`：范围查询（`type=range`）

例如联合索引如下，使用三个索引列查询时，执行计划如下（注意 `key_len` 和 `ref`）：

```sql
`channel_task_no` varchar(60) NOT NULL,
`reconciliation_code` tinyint(4) unsigned NOT NULL DEFAULT '0',
`reconciliation_status` tinyint(4) unsigned NOT NULL DEFAULT '0',
KEY `idx_taskno_rcode_rstatus` (`channel_task_no`,`reconciliation_code`,`reconciliation_status`) USING BTREE

+----+-------------+-------+------+--------------------------+--------------------------+---------+-------------------+------+-----------------------+
| id | select_type | table | type | possible_keys            | key                      | key_len | ref               | rows | Extra                 |
+----+-------------+-------+------+--------------------------+--------------------------+---------+-------------------+------+-----------------------+
|  1 | SIMPLE      | t_xxx | ref  | idx_taskno_rcode_rstatus | idx_taskno_rcode_rstatus | 184     | const,const,const |    1 | Using index condition |
+----+-------------+-------+------+--------------------------+--------------------------+---------+-------------------+------+-----------------------+
```

# rows

表示 MySQL 认为执行查询必须扫描的行数。

对于 InnoDB 表，此数字是估计值，可能并不总是准确。

当 `prossible_keys` 存在多个可选索引时，优化器会选择一个认为最优的执行方案，以最小的代价去执行语句。其中，这个扫描行数就是影响执行代价的因素之一。扫描的行数越少，意味着访问磁盘数据的 IO 次数越少，消耗的 CPU 资源也越少。

当然，扫描行数并不是唯一的判断标准，优化器还会结合是否使用临时表、是否排序等因素进行综合判断。

所以在实践中，如果你发现 explain 的结果预估的 `rows` 值跟实际情况差距比较大，可以采用执行 `analyze table` 重新统计信息。

# Extra

这一列显示的是额外信息。如果想要查询越快越好，需要特别留意 `Extra` 列是否出现以下情况：

| Extra                                   | 缓冲区        | 大小配置                                                     | 数据结构                            | 备注                                                         |
| --------------------------------------- | ------------- | ------------------------------------------------------------ | ----------------------------------- | ------------------------------------------------------------ |
| `Using filesort`                        | `sort_buffer` | `sort_buffer_size`                                           | 有序数组                            | 使用了“外部排序”（全字段排序或 rowid 排序）                  |
| `Using join buffer (Block Nested Loop)` | `join_buffer` | `join_buffer_size`                                           | 无序数组                            | 使用了“基于块的嵌套循环连接”算法（Block Nested-Loop Join（BNL）） |
| `Using temporary`                       | 临时表        | 小于 `tmp_table_size` 为内存临时表，否则为磁盘临时表（可以使用 `SQL_BIG_RESULT` 直接指定） | 二维表结构（类似于 Map，Key-Value） | 如果执行逻辑需要用到二维表特性，就会优先考虑使用临时表。例如：`DISTINCT`、`GROUP BY`、`UNION` |

这三个数据结构都是用来存放 `SELECT` 语句执行过程中的中间数据，以辅助 SQL 语句的执行的。这些情况通常都能通过索引优化。

各种常见的重要值如下：

## Using index

使用了覆盖索引。

## Using where

使用 `WHERE` 条件过滤结果，但查询的列未被索引覆盖。

## Using index condition

查询的列不完全被索引覆盖。

例如：[索引下推优化（ICP）](https://dev.mysql.com/doc/refman/5.7/en/index-condition-pushdown-optimization.html)

## Using temporary

MySQL 需要创建一张临时表来处理查询。通常发生于查询包含 `DISTINCT`、`GROUP BY` 或 `ORDER BY` 子句等需要数据去重的场景。出现这种情况一般是要进行优化的，首先想到的是用索引进行优化。

## Using join buffer

使用 BNL 算法进行表连接。这种情况下一般考虑使用索引对被驱动表的表连接字段进行优化，以使用更高效的 NLJ 算法。

## Using filesort

将用外部排序而不是索引排序，数据较少时从内存排序，否则需要在磁盘完成排序。这种情况下一般考虑使用索引进行优化。

优化参考：[Section 8.2.1.14, “ORDER BY Optimization”](https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html)

# 参考

https://dev.mysql.com/doc/refman/5.7/en/execution-plan-information.html

https://dev.mysql.com/doc/refman/5.7/en/analyze-table.html

*MySQL Workbench* has a Visual Explain capability that provides a visual representation of [EXPLAIN](https://dev.mysql.com/doc/refman/5.7/en/explain.html) output. See [Tutorial: Using Explain to Improve Query Performance](https://dev.mysql.com/doc/workbench/en/wb-tutorial-visual-explain-dbt3.html).