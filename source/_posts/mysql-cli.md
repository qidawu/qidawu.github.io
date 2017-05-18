---
title: MySQL 常用命令
date: 2017-04-30 15:16:33
updated:
tags: MySQL
---

高安全的生产环境下只能使用命令行操作数据库，下面介绍一些常用命令。

# 连接 DB

```
$ mysql -h192.168.0.221 -P3306 -u账号 -p密码 [db_name]

or better:

$ mycli -h192.168.0.221 -P3306 -u账号 -p密码 [db_name]
```

# 查看库

```
// 查看所有库
$ show databases;

+------------------+
| Database         |
|------------------|
| db_name_1        |
| db_name_2        |
+------------------+

// 进入某个库
$ use db_name_1;
```

# 查看表

## 所有表

```
$ show tables [from db_name];

+---------------------+
| Tables_in_db_name   |
|---------------------|
| table_name_1        |
| table_name_2        |
+---------------------+
```

## 所有表状态

显示当前使用或者指定的 DB 中的每个表的信息。

由于字段较多，可用 `\G` 参数按列显示（行转列），起到显示美化的作用，方便查看：

```
$ show table status [from db_name] \G;

Name            | table_name
Engine          | InnoDB
Version         | 10
Row_format      | Compact
Rows            | 59079
Avg_row_length  | 133
Data_length     | 7880704
Max_data_length | 0
Index_length    | 21069824
Data_free       | 5242880
Auto_increment  | 75437
Create_time     | 2017-04-13 20:51:55
Update_time     | None
Check_time      | None
Collation       | utf8_general_ci
Checksum        | None
Create_options  |
Comment         | 测试表
```

比较重要的字段：

| 字段               | 描述                                       |
| ---------------- | ---------------------------------------- |
| `Rows`           | 行的数目。部分存储引擎，如 `MyISAM`，存储精确的数目。<br/>对于其它存储引擎，比如 `InnoDB`，是一个大约的值，与实际值相差可达40到50％。在这些情况下，使用 `SELECT COUNT(*)` 来获得准确的数目。 |
| `Avg_row_length` | 平均的行长度。                                  |
| `Data_length`    | 对于 `MyISAM`，`Data_length` 是数据文件的长度（以字节为单位）。<br/>对于 `InnoDB`，`Data_length` 是聚簇索引 `clustered index` 大约分配的内存量（以字节为单位）。 |
| `Index_length`   | 对于 `MyISAM`，`Index_length` 是索引文件的长度（以字节为单位）。<br/>对于 `InnoDB`，`Index_length` 是非聚簇索引 `non-clustered index` 大约分配的内存量（以字节为单位）。 |
| `Auto_increment` | 下一个 `AUTO_INCREMENT` 值。                  |

## 表结构

查看列名（三者等价）：

```
$ show columns from table_name [from db_name];
$ show columns from [db_name.]table_name;
$ desc table_name;  // 简写形式
```

## 索引

```
$ show index from table_name;
```

## 建表语句

```
$ show create table table_name;
```

# 查看用户权限

显示一个用户的权限，显示结果类似于 `GRANT` 命令：

```
$ show grants [for user_name@'192.168.0.%'];

+---------------------------------------------------------------------------------------------+
| Grants for user_name@192.168.0.%                                                            |
+---------------------------------------------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE ON `db_name`.`table_name` TO 'user_name'@'192.168.0.%' |
+---------------------------------------------------------------------------------------------+
```

# 查看系统相关

## 系统状态

显示一些系统特定资源的信息，例如，正在运行的线程数量。

```
$ show status;
```

## 系统变量

显示系统变量的名称和值。

```
$ show variables;
```

## DB 进程

显示系统中正在运行的所有进程，也就是当前正在执行的查询。大多数用户可以查看他们自己的进程，但是如果他们拥有process权限，就可以查看所有人的进程，包括密码。

```
// 查看当前 DB 进程
$ show processlist;
$ show full processlist;
+----------+-----------+--------------------+---------+---------+------+-------+------------------+
| Id       | User      | Host               | db      | Command | Time | State | Info             |
+----------+-----------+--------------------+---------+---------+------+-------+------------------+
| 33702451 | user_name | 192.168.0.200:49764 | db_name | Query   |    0 | init  | show processlist |
+----------+-----------+--------------------+---------+---------+------+-------+------------------+
```

| 字段        | 描述                                       |
| --------- | ---------------------------------------- |
| `Id`      | 标识，用途：`kill 33702451 ` 杀死指定进程。           |
| `User`    | 显示执行 SQL 的用户。                            |
| `Host`    | 显示这个账号是从哪个 IP 连过来的。                      |
| `db`      | 显示这个进程目前连接的是哪个数据库 。                      |
| `command` | 显示当前连接的执行命令，一般就是休眠（ sleep ），查询（ query ），连接（ connect ）。 |
| `Time`    | 这个状态持续的时间，单位是秒。                          |
| `State`   | 显示使用当前连接的 SQL 语句的状态。                     |
| `Info`    | 显示执行的 SQL 语句。                            |

## 权限

显示服务器所支持的不同权限。

```
$ show privileges;
```

## 存储引擎

```
$ show engies; // 显示安装以后可用的存储引擎和默认引擎。 

$ show innodb status; // 显示innoDB存储引擎的状态 

$ show logs; // 显示BDB存储引擎的日志 
```

## 警告与错误

```
$ show warnings; // 显示最后一个执行的语句所产生的错误、警告和通知 

$ show errors; // 只显示最后一个执行语句所产生的错误
```
