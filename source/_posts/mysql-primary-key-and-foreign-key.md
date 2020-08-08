---
title: MySQL 主键与外键总结
date: 2020-01-03 15:25:31
updated:
tags: MySQL
typora-root-url: ..
---

# 定义

MySQL 原生支持外键（即允许跨表交叉引用相关数据）和外键约束（用于保持**数据一致性**！）。

外键关系涉及包含初值的父表，以及引用父表值的子表。而外键约束就定义在子表之上。

> A foreign key relationship involves a **parent table** that holds the initial column values, and a **child table** with column values that reference the parent column values. A foreign key constraint is defined on the child table.

# 语法

在 `CREATE TABLE` 或 `ALTER TABLE` 语句中定义外键约束的基本语法如下：

```SQL
[CONSTRAINT [fk_symbol]] FOREIGN KEY
    [index_name] (col_name, ...)
    REFERENCES tbl_name (col_name,...)
    [ON DELETE reference_option]
    [ON UPDATE reference_option]

reference_option:
    RESTRICT | CASCADE | SET NULL | NO ACTION | SET DEFAULT
```

删除外键约束：

```SQL
ALTER TABLE tbl_name DROP FOREIGN KEY fk_symbol;
```

# 例子

```SQL
# 创建父表
CREATE TABLE `t_parent` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='父表';

# 创建子表
CREATE TABLE `t_child` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `parent_id` int(10) unsigned NOT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_parent_id` (`parent_id`),
  CONSTRAINT `fk_parent_id` FOREIGN KEY (`parent_id`) REFERENCES `t_parent` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='子表';
```

可视化界面如下：

![foreign_key_constraint](/img/mysql/foreign_key_constraint.png)

注意：

* 创建外键约束时，如果主外键之间的数据类型不一致（例如长度、无符号），会报错：`1215 - Cannot add the foreign key constraint`。

* 创建外键约束后，MySQL 会为子表自动创建普通索引 `fk_parent_id`，以提升 `join` 查询性能。

* 创建外键不一定只能引用父表的主键，也能引用普通列。如果引用普通列，MySQL 则会在父表和子表同时为该列创建普通索引。如果删除该索引会报错：`1553 - Cannot drop index '...': needed in a foreign key constraint`。

* `reference_option` 的几种情况总结如下：

  * 操作父表：
  
    * `RESTRICT` 在 `UPDATE` 或者 `DELETE` 父表记录时，对子表进行**一致性检查**。
    * `CASCADE` 在 `UPDATE` 或者 `DELETE` 父表记录时，对子表进行**级联操作**。
    * `SET NULL` 在 `UPDATE` 或者 `DELETE` 父表记录时，对子表进行 **SET NULL 操作**。
  
    |          | `RESTRICT` (`NO ACTION`)                                     | `CASCADE`                  | `SET NULL`                  |
  | -------- | ------------------------------------------------------------ | -------------------------- | --------------------------- |
    | `INSERT` | 正常插入                                                     | 正常插入                   | 正常插入                    |
    | `UPDATE` | 更新父表值，会报错 `1451 - Cannot delete or update a parent row: a foreign key constraint fails` | 更新父表值，子表值级联更新 | 更新父表值，子表值 SET NULL |
    | `DELETE` | 删除父表行，会报错 `1451 - Cannot delete or update a parent row: a foreign key constraint fails` | 删除父表行，子表行级联删除 | 删除父表行，子表值 SET NULL |
    
  * 操作子表：
  
    * `INSERT`、`UPDATE` 触发一致性检查。
    
    |          | `RESTRICT` (`NO ACTION`)                                     | `CASCADE` | `SET NULL` |
    | -------- | ------------------------------------------------------------ | --------- | ---------- |
    | `INSERT` | 无论哪个 option，插入子表行为父表不存在的值，都会报错 `1452 - Cannot add or update a child row: a foreign key constraint fails` |           |            |
  | `UPDATE` | 同上                                                         |           |            |
    | `DELETE` | 无论哪个 option，删除子表行都 ok                             |           |            |

# 总结

![primary_key_and_foreign_key](/img/mysql/primary_key_and_foreign_key.png)

# 参考

https://dev.mysql.com/doc/refman/5.7/en/create-table-foreign-keys.html

https://mp.weixin.qq.com/s/jOF1rohb6OvA3Pb5rL6Ilg