---
title: MySQL SELECT 语法总结
date: 2018-02-02 23:37:58
updated:
tags: MySQL
---

# 基础语法

`SELECT` 基础语法如下：

```mysql
SELECT
    [ALL | DISTINCT]
    select_expr [, select_expr ...]
    [
      FROM table_references
      [WHERE where_condition]
      [GROUP BY {col_name | expr | position}
        [ASC | DESC], ... [WITH ROLLUP]]
      [HAVING where_condition]
      [ORDER BY {col_name | expr | position}
        [ASC | DESC], ...]
      [LIMIT {[offset,] row_count | row_count OFFSET offset}]
      [
        INTO OUTFILE 'file_name'
          [CHARACTER SET charset_name]
          export_options
        | INTO DUMPFILE 'file_name'
        | INTO var_name [, var_name]
      ]
      [FOR UPDATE | LOCK IN SHARE MODE]
    ]
```

`SELECT` 语句可用于检索单个、多个、所有列（星号 `*` 通配符）。每个 *select_expr* 表示您想要检索的列。必须至少有一个 *select_expr*。

## 去重

修饰符 `ALL` 和 `DISTINCT` 用于指定**重复行是否应该返回（是否去重）**，作用于所有的列，而不仅仅是跟在其后的那一列。例如 `SELECT DISTINCT vend_id, prod_price` ，除非指定的两列完全相同，否则所有的行都会被检索出来。

| 修饰符        | 描述                     |
| ---------- | ---------------------- |
| ` ALL`     | 默认值，指定应返回所有匹配的行，包括重复项。 |
| `DISTINCT` | 指定从结果集中删除重复的行。         |

## 检索表

*table_references* 指示检索表，其语法可参考 [JOIN语法](https://dev.mysql.com/doc/refman/5.7/en/join.html)。`SELECT` 也可以不使用 `FROM` 子句而用来检索计算出的行：

```mysql
SELECT 1 + 1;
 -> 2
```

也可以使用 `FROM DUAL` 指定虚拟表，MySQL 会忽略这个子句。

```mysql
SELECT 1 + 1 FROM DUAL;
 -> 2
```

## 过滤数据

`WHERE` 子句用于过滤数据，*where_condition* 可以使用以下表达式语法：

```mysql
expr:
    expr OR expr
  | expr || expr
  | expr XOR expr
  | expr AND expr
  | expr && expr
  | NOT expr
  | ! expr
  | boolean_primary IS [NOT] {TRUE | FALSE | UNKNOWN}
  | boolean_primary

boolean_primary:
    boolean_primary IS [NOT] NULL
  | boolean_primary <=> predicate
  | boolean_primary comparison_operator predicate
  | boolean_primary comparison_operator {ALL | ANY} (subquery)
  | predicate

comparison_operator: = | >= | > | <= | < | <> | !=

predicate:
    bit_expr [NOT] IN (subquery)
  | bit_expr [NOT] IN (expr [, expr] ...)
  | bit_expr [NOT] BETWEEN bit_expr AND predicate
  | bit_expr SOUNDS LIKE bit_expr
  | bit_expr [NOT] LIKE simple_expr [ESCAPE simple_expr]
  | bit_expr [NOT] REGEXP bit_expr
  | bit_expr

bit_expr:
  ...

simple_expr:
  ...
```

`LIKE` 可使用以下通配符：

| 通配符  | 描述      |
| ---- | ------- |
| `%`  | 匹配多个字符  |
| `_`  | 匹配单个字符  |
| `[]` | 匹配一个字符集 |

## 排序数据

`ORDER BY` 子句用于排序，使用以下关键字进行升序或降序排序，要注意关键字只应用于直接位于其前面的列名。如果想在多个列上进行降序排序，必须对每一列指定 `DESC` 关键字。

| 关键字 | 描述             |
| ------ | ---------------- |
| `ASC`  | 升序排序（默认） |
| `DESC` | 降序排序         |

## 限制结果集

`LIMIT` 子句可用于限制 `SELECT` 语句返回的结果集：

```mysql
SELECT * FROM tbl LIMIT 5;     # Retrieve first 5 rows, equivalent to LIMIT 0, 5
SELECT * FROM tbl LIMIT 5,10;  # Retrieve rows 6-15
SELECT * FROM tbl LIMIT 5 OFFSET 10;  # Retrieve rows 11-15
```

## 汇总数据

我们经常需要汇总数据而不用把它们实际检索出来，为此 SQL 提供了五个聚集函数（aggregate function）。使用这些函数，SQL 查询可用于检索数据，以便分析和报表生成。这种类型的检索例子有：

* 确定表中行数（或者满足某个条件或包含某个特定值的行数）；
* 获得表中某些行的和；
* 找出表列（或所有行或某些特定的行）的最大值、最小值、平均值。

| 聚集函数      | 描述                                       |
| --------- | ---------------------------------------- |
| `COUNT()` | 返回某列的行数，忽略 `NULL` 值。`COUNT(*)` 则包含 `NULL` 值。 |
| `AVG()`   | 返回某列的平均值，忽略 `NULL` 值。                    |
| `MAX()`   | 返回某列的最大值，忽略 `NULL` 值。                    |
| `MIN()`   | 返回某列的最小值，忽略 `NULL` 值。                    |
| `SUM()`   | 返回某列值之和，忽略 `NULL` 值。                     |

修饰符 `ALL` 和 `DISTINCT` 可用于指定重复行是否应该返回。例如：

```mysql
# 返回特定供应商所提供产品的平均价格
SELECT AVG(prod_price) AS avg_price
FROM Products
WHERE vend_id = 'DLL01';
 -> 3.8650

# 同上，但平均值只考虑各个不同的价格
SELECT AVG(DISTINCT prod_price) AS avg_price
FROM Products
WHERE vend_id = 'DLL01';
 -> 4.2400
```

可以看到，使用了 `DISTINCT` 后的 *avg_price* 会比较高，因为此例子中有多个物品具有相同的较低价格，排除它们提升了平均价格。

`SELECT` 语句也可根据需要同时使用多个聚集函数：

```mysql
SELECT COUNT(*) AS num_items,
       MIN(prod_price) AS price_min,
       MAX(prod_price) AS price_max,
       AVG(prod_price) AS price_avg
FROM Products;
```

## 分组数据

`GROUP BY` 子句用于分组数据，注意：

*   `GROUP BY` 子句可以包含任意数目的列，因而可以对分组进行嵌套，更细致地进行数据分组。
*   如果在 `GROUP BY` 子句中嵌套了分组，数据将在最后指定的分组上进行汇总。换句话说，在建立分组时，指定的所有列都一起计算（所以不能从个别的列取回数据）。
*   `GROUP BY` 子句中列出的每一列都必须是检索列或有效的表达式（但不能是聚集函数）。如果在 `SELECT` 中使用表达式，则必须在 `GROUP BY` 子句中指定相同的表达式。不能使用别名。
*   除聚集计算语句外，`SELECT` 语句中的每一列都必须在 `GROUP BY` 子句中给出。否则，如果 `SELECT ` 语句中出现了 `GROUP BY ` 中没有的列，假如该分组内的条目数大于 1，这样的列显示的内容为第一个条目的值。
*   如果分组列中包含具有 `NULL` 值的行，则 `NULL` 将作为一个分组返回。如果列中有多行 `NULL` 值，它们将分为一组。

`GROUP BY` 可以搭配使用 `HAVING` 过滤分组。`HAVING` 和 `WHERE` 的差别在于，`WHERE` 对分组前的数据进行过滤， `HAVING` 对分组后的数据进行过滤。

# 加锁读

`InnoDB` 支持两种类型的 [加锁读（Locking Reads）](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_locking_read)，为事务操作提供额外的**安全性**：

* `SELECT ... LOCK IN SHARE MODE ` 设置**共享（*S*）锁**
*  `SELECT ... FOR UPDATE` 设置**排它（*X*）锁**

详情请参考《[MySQL 锁机制总结](/2018/10/20/mysql-locking/)》。

# UNION 子句

语法：

```SQL
SELECT ...
UNION [ALL | DISTINCT] SELECT ...
[UNION [ALL | DISTINCT] SELECT ...]
```

`UNION` 将来自多个 `SELECT` 语句的结果合并为一个结果集，结果集列名取自第一条 `SELECT` 语句的列名。

`UNION` 或 `UNION DISTINCT` 去重，而 `UNION ALL` 则不去重。

要为单独的一条 `SELECT` 语句应用 `ORDER BY` 或 `LIMIT` 子句，需要将子句放在包含 `SELECT` 的括号内，例如：

```SQL
(SELECT a FROM t1 WHERE a=10 AND B=1 ORDER BY a LIMIT 10)
UNION
(SELECT a FROM t2 WHERE a=11 AND B=2 ORDER BY a LIMIT 10);
```

反之，如要应用到整个 `UNION` 结果集，则在单个 `SELECT` 语句后面加上括号，并在最后一个语句后面加上子句，例如：

```SQL
(SELECT a FROM t1 WHERE a=10 AND B=1)
UNION
(SELECT a FROM t2 WHERE a=11 AND B=2)
ORDER BY a LIMIT 10;
```



# 参考

《MySQL 必知必会》

https://dev.mysql.com/doc/refman/5.7/en/select.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html

