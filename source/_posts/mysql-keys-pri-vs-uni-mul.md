---
title: MySQL 索引类型 PRI、UNI、MUL 的区别
date: 2017-04-30 18:55:53
updated:
tags: MySQL
---

`DESC` 命令查看表结构时，索引列 `Key` 共有三种类型：`PRI`、`UNI`、`MUL`

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

参考：《[SHOW COLUMNS Syntax](https://dev.mysql.com/doc/refman/5.7/en/show-columns.html)》