---
title: MySQL 字符集与排序规则总结
date: 2019-01-03 22:29:12
updated:
tags: MySQL
typora-root-url: ..
---

查看数据库支持的字符集：`SHOW CHARACTER SET`

查看数据库支持的排序规则：`SHOW COLLATION`

# 字符集（Character Sets）

下表是 MySQL 中八个可能影响到字符集的系统变量，其中有几个如果配置不当可能会乱码问题，需重点关注：

| 变量                       | 默认值   | 描述                                                         |
| -------------------------- | -------- | ------------------------------------------------------------ |
| `character_set_client`     | `utf8`   | The character set for statements that arrive from the client. |
| `character_set_connection` | `utf8`   | The character set used for literals specified without a character set introducer and for number-to-string conversion. |
| `character_set_database`   | `latin1` | The character set used by the default database.              |
| `character_set_filesystem` | `binary` | The file system character set.                               |
| `character_set_results`    | `utf8`   | The character set used for returning query results to the client. This includes result data such as column values, result metadata such as column names, and error messages. |
| `character_set_server`     | `latin1` | The server's default character set.                          |
| `character_set_system`     | `utf8`   | The character set used by the server for storing identifiers. |
| `character_sets_dir`       |          | The directory where character sets are installed.            |

可以通过下图来了解 MySQL 内部字符集转换过程：

![MySQL Character Set](/img/mysql/mysql_character_set.jpg)

1. MySQL 收到请求时将请求数据从 `character_set_client` 转换为 `character_set_connection`
2. 进行内部操作前将请求数据从 `character_set_connection` 转换为内部操作字符集，步骤如下：
   1. 使用每个数据字段的 `CHARACTER SET` 设定值；
   2. 若上述值不存在，则使用对应数据表的字符集设定值；
   3. 若上述值不存在，则使用对应数据库的字符集设定值；
   4. 若上述值不存在，则使用 `character_set_server` 设定值。
3. 最后将操作结果从内部操作字符集转换为 `character_set_results`

而系统变量 `character_set_database` 主要用来设置默认创建数据库的编码格式，如果在创建数据库时没有设置编码格式，就按照这个格式设置，如下：

```sql
CREATE DATABASE `testdb` /*!40100 DEFAULT CHARACTER SET latin1 */
```

从而影响到建表时默认的字符集：

```sql
CREATE TABLE `test` (
  `name` varchar(255) NOT NULL DEFAULT ''
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

从而影响到中文字符的插入：

```sql
INSERT INTO test values ('你好');

[Err] 1366 - Incorrect string value: '\xE4\xBD\xA0\xE5\xA5\xBD' for column 'name' at row 1
```

## 配置说明

```
[mysqld]
# 影响系统变量 character_set_database 和 character_set_server
character-set-server = utf8
collation-server = utf8_unicode_ci
init-connect = 'SET NAMES utf8'

# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB

[client]
default-character-set = utf8

[mysql]
default-character-set = utf8
```

配置后，需要重启服务：

```
net stop mysql
net start mysql
```

之后，通过命令 `SHOW VARIABLES LIKE '%character%'` 查看结果：

```
+--------------------------+----------+
| Variable_name            | Value    |
+--------------------------+----------+
| character_set_client     | utf8     |
| character_set_connection | utf8     |
| character_set_database   | utf8     |
| character_set_filesystem | binary   |
| character_set_results    | utf8     |
| character_set_server     | utf8     |
| character_set_system     | utf8     |
| character_sets_dir       | ...      |
+--------------------------+----------+
```

如果在创建数据库之前，没有在配置文件中配置好默认字符集，可以通过 `SET` 命令进行修改。

配置好后，建库结果如下：

```sql
CREATE DATABASE `testdb` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci */
```

# 排序规则（Collations）

# 参考

https://dev.mysql.com/doc/refman/5.7/en/charset.html

https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html

![charset](/img/mysql/charset.png)