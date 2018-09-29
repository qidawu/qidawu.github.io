---
title: SELECT 语法总结
date: 2017-08-24 23:37:58
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

## 检索不同的值

修饰符 `ALL` 和 `DISTINCT` 用于指定重复行是否应该返回，作用于所有的列，而不仅仅是跟在其后的那一列。例如 `SELECT DISTINCT vend_id, prod_price` ，除非指定的两列完全相同，否则所有的行都会被检索出来。

| 修饰符        | 描述                     |
| ---------- | ---------------------- |
| ` ALL`     | 默认值，指定应返回所有匹配的行，包括重复项。 |
| `DISTINCT` | 指定从结果集中删除重复的行。         |

## 限制结果集

`LIMIT` 子句可用于限制 `SELECT` 语句返回的结果集：

```mysql
SELECT * FROM tbl LIMIT 5;     # Retrieve first 5 rows
SELECT * FROM tbl LIMIT 5,10;  # Retrieve rows 6-15
SELECT * FROM tbl LIMIT 5 OFFSET 10;  # Retrieve rows 11-15
```

## 排序数据

`ORDER BY` 子句用于排序，使用以下关键字进行升序或降序排序，要注意关键字只应用于直接位于其前面的列名。如果想在多个列上进行降序排序，必须对每一列指定 `DESC` 关键字。

| 关键字    | 描述       |
| ------ | -------- |
| `ASC`  | 升序排序（默认） |
| `DESC` | 降序排序     |

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
*   除聚集计算语句外，`SELECT` 语句中的每一列都必须在 `GROUP BY` 子句中给出。
*   如果分组列中包含具有 `NULL` 值的行，则 `NULL` 将作为一个分组返回。如果列中有多行 `NULL` 值，它们将分为一组。
*   可以使用 `HAVING` 过滤分组。`HAVING` 和 `WHERE` 的差别在于，`WHERE` 在数据分组前进行过滤， `HAVING` 在数据分组后进行过滤。

## 表联结

### 内联结

```mysql
# 简单的等值语法创建内联结
SELECT vend_name, prod_name, prod_price
FROM Vendors, Products
WHERE Vendors.vend_id = Products.vend_id;

# ANSI SQL 规范首选 INNER JOIN 语法创建内联结
SELECT vend_name, prod_name, prod_price
FROM Vendors INNER JOIN Products
ON Vendors.vend_id = Products.vend_id;
```

在内联结两个表时，实际要做的是将第一个表中的每一行与第二个表中的每一行配对，`WHERE` 或 `ON` 子句作为过滤条件，**只包含那些匹配联结条件的行**。

由没有联结条件的表关系返回的结果为**笛卡儿积（cartesian product）**。检索出的行的数目将是第一个表中的行数乘以第二个表中的行数。因此应当总是提供联结条件。

### 外联结（左、右）

许多联结将一个表中的行与另一个表中的行相关联，但有时候需要**包含没有关联行**的那些行，例如：

*   对每个顾客下的订单进行计数，包括那些至今尚未下订单的顾客；

*   列出所有产品以及订购数量，包括没有人订购的产品；

*   计算平均销售规模，包括那些至今尚未下订单的顾客。

在上述例子中，联结包含了那些在相关表中没有关联行的行。这种联结称为外联结。

例如，要检索出所有顾客+订单，包括那些还未下单的顾客：

```mysql
SELECT Customers.cust_id, Orders.order_num
FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;

->
cust_id    order_num
---------- ---------
1000000001 20005
1000000001 20009
1000000002 NULL
1000000003 20006
1000000004 20007
1000000005 20008
```

上例如果使用内联结，将不包含 *1000000002* 顾客，因为他还未下单（即联结条件不匹配）。

作为对比，下例使用内联结 `INNER JOIN` 和聚集函数 `COUNT()` 统计出所有顾客的订单数：

```mysql
SELECT Customers.cust_id, COUNT(Orders.order_num) AS num_ord
FROM Customers INNER JOIN Orders
ON Customers.cust_id = Orders.cust_id
GROUP BY Customers.cust_id;

->
cust_id    num_ord
---------- --------
1000000001 2
1000000003 1
1000000004 1
1000000005 1
```

但如果使用左外联结 `LEFT OUTER JOIN` 和聚集函数 `COUNT()` 进行相同统计，将会包括那些还未下单的顾客，例如顾客 *1000000002*：

```mysql
SELECT Customers.cust_id, COUNT(Orders.order_num) AS num_ord
FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id
GROUP BY Customers.cust_id;

->
cust_id    num_ord
---------- -------
1000000001 2
1000000002 0
1000000003 1
1000000004 1
1000000005 1
```

由于 `COUNT(column)` 计数会忽略 `NULL` 值，因此顾客 *1000000002* 的统计结果为 0。

注意，左、右外联结之间的唯一差别是所关联的表的顺序。换句话说，调整 `FROM` 或 `WHERE` 子句中表的顺序，左外联结可以转换为右外联结。因此，这两种外联结可以互换使用，哪个方便就用哪个。

