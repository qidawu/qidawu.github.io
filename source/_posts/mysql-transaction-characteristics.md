---
title: MySQL 事务实操总结
date: 2018-10-26 22:39:42
updated:
tags: MySQL
---

前文总结了 MySQL 事务的一些概念，下面总结下如何进行实操。

# 开启事务、提交与回滚

```sql
START TRANSACTION
    [transaction_characteristic [, transaction_characteristic] ...]

transaction_characteristic: {
    WITH CONSISTENT SNAPSHOT
  | READ WRITE
  | READ ONLY
}

BEGIN
COMMIT
ROLLBACK
SET autocommit = {0 | 1}
```

主要语法作用如下：

* `START TRANSACTION` 或 `BEGIN` 开启新的事务。
* `COMMIT` 提交当前事务，使其更改持久化。
* `ROLLBACK` 回滚当前事务，取消其更改。
* `SET autocommit` 禁用或启用当前会话的默认自动提交模式。

`START TRANSACTION` 是标准的 SQL 语法，推荐使用。它支持以下 `BEGIN` 语法所不支持的修饰符：

* `READ WRITE` 读写模式，默认值。
* `READ ONLY` 只读模式，有助于提升存储引擎的性能表现。

# SET TRANSACTION 语法

可以通过 [SET TRANSACTION](https://dev.mysql.com/doc/refman/5.7/en/set-transaction.html) 语句设置事务的特性，包括隔离级别和读写模式：

```sql
SET [GLOBAL | SESSION] TRANSACTION
    transaction_characteristic [, transaction_characteristic] ...

transaction_characteristic: {
    ISOLATION LEVEL level
  | access_mode
}

level: {
     REPEATABLE READ
   | READ COMMITTED
   | READ UNCOMMITTED
   | SERIALIZABLE
}

access_mode: {
     READ WRITE
   | READ ONLY
}
```

## 事务特性范围（作用域）

您可以设置事务特性的作用域为全局、当前会话或仅针对下一个事务，其优先级为事务 > 会话 > 全局：

- 使用 `GLOBAL` 关键字：
  - 全局应用于所有后续会话。
  - 现有会话不受影响。
  - 全局设置要求 [SUPER](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_super) 权限。

- 使用 `SESSION` 关键字：
  - 应用于当前会话中执行的所有后续事务。
  - 不影响正在进行的事务。

- 没有 `SESSION` 或  `GLOBAL` 关键字：

  - 仅应用于当前会话中执行的下一个事务。

  - 后续事务将恢复为当前会话的默认值。

  - 事务中不允许使用该语句：

    ```sql
    START TRANSACTION;
    SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    ERROR 1568 (25001): Transaction characteristics can't be changed while a transaction is in progress
    ```

语法总结如下：

| 语法                                                 | 作用域                |
| ---------------------------------------------------- | --------------------- |
| SET GLOBAL TRANSACTION *transaction_characteristic*  | Global                |
| SET SESSION TRANSACTION *transaction_characteristic* | Session               |
| SET TRANSACTION *transaction_characteristic*         | Next transaction only |

## 事务隔离级别

MySQL 能够识别所有的四个事务隔离级别，`InnoDB` 引擎也支持所有的隔离级别。可以使用 `ISOLATION LEVEL level` 子句进行设置：

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; --读未提交
SET TRANSACTION ISOLATION LEVEL READ COMMITTED; --读已提交
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ; --可重复读
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE; --串行化
```

## 事务读写模式

MySQL 支持两种事务读写模式，其设置方式如下：

```sql
SET TRANSACTION READ WRITE; --读写模式，默认值
SET TRANSACTION READ ONLY; --只读模式，有助于提升存储引擎的性能表现
```

如果要单独为某个事务指定读写模式，搭配 `START TRANSACTION` 使用。

# SET 语法

也可以通过 [`SET`](https://dev.mysql.com/doc/refman/5.7/en/set-variable.html) 语句直接进行各种变量赋值，语法总结如下：

| 语法                               | 作用域                |
| ---------------------------------- | --------------------- |
| SET GLOBAL *var_name* = *value*    | Global                |
| SET @@GLOBAL.*var_name* = *value*  | Global                |
| SET SESSION *var_name* = *value*   | Session               |
| SET @@SESSION.*var_name* = *value* | Session               |
| SET *var_name* = *value*           | Session               |
| SET @@*var_name* = *value*         | Next transaction only |

变量的查询语法如下，例如 [`transaction_isolation`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_transaction_isolation) 和 [`transaction_read_only`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_transaction_read_only) ：

```sql
SELECT @@GLOBAL.transaction_isolation, @@GLOBAL.transaction_read_only;
SELECT @@SESSION.transaction_isolation, @@SESSION.transaction_read_only;
```

# 启动时设置

上面介绍的两种语法都是用于运行时设置，下面介绍两种方式用于在服务启动时设置：

## 命令行参数

```
--transaction-isolation=REPEATABLE-READ
--transaction-read-only=OFF
```

## 配置文件

```
[mysqld]
transaction-isolation = REPEATABLE-READ
transaction-read-only = OFF
```

# 参考

https://dev.mysql.com/doc/refman/5.7/en/commit.html

https://dev.mysql.com/doc/refman/5.7/en/set-transaction.html

https://dev.mysql.com/doc/refman/5.7/en/set-variable.html

https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-ro-txn.html