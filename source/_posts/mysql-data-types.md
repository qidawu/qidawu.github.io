---
title: MySQL 常用数据类型
date: 2018-01-05 22:27:21
updated:
tags: MySQL
typora-root-url: ..
---

# 原则

MySQL 支持的数据类型非常多，选择正确的数据类型对于获得高性能至关重要。不管存储哪种类型的数据，下面几个简单的原则都有助于作出更好的选择。

## Smaller is usually better

更小的通常更好：

> In general, try to use the smallest data type that can correctly store and represent your data. Smaller data types are usually faster, because they use less space on the disk, in memory, and in the CPU cache. They also generally require fewer CPU cycles to process.
>
> Make sure you don’t underestimate the range of values you need to store, though, because increasing the data type range in multiple places in your schema can be a painful and time-consuming operation. If you’re in doubt as to which is the best data type to use, choose the smallest one that you don’t think you’ll exceed. (If the system is not very busy or doesn’t store much data, or if you’re at an early phase in the design process, you can change it easily later.)

更小的数据类型通常更快，因为占用更少的磁盘、内存和 CPU 缓存，并且处理时需要的 CPU 周期也更少。

## Simple is good

简单就好：

> Fewer CPU cycles are typically required to process operations on simpler data types. For example, integers are cheaper to compare than characters, because character sets and collations (sorting rules) make character comparisons complicated. Here are two examples: you should store dates and times in MySQL’s builtin types instead of as strings, and you should use integers for IP addresses. We discuss these topics further later.

简单数据类型的操作通常需要更少的 CPU 周期、索引性能更好。例如，整型比字符操作代价更低，因为字符集和校对规则（排序规则）使得字符比整型更复杂。

有几个例子：

* 使用日期与时间类型，而不是字符串来存储日期和时间，以便排序和格式转换。

* IPv4 地址：按十进制标记法（32-bit decimal notation）使用 32 位无符号整型类型（`int unsigned`）进行存储，而不是按点分十进制标记法（Dot-decimal notation）使用字符串类型（`varchar`）进行存储。

  ```sql
  -- 测试表（注意使用 int UNSIGNED 类型存储（存储范围：0 ~ 2^32-1））
  CREATE TABLE `t_iptable` (
    `ip1` int(32) UNSIGNED NOT NULL COMMENT 'IPv4 地址',
    `ip2` varchar(15) NOT NULL COMMENT 'IPv4 地址'
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  
  -- 测试数据
  INSERT INTO t_iptable(ip1, ip2) VALUES(INET_ATON('0.0.0.0'), '0.0.0.0');
  INSERT INTO t_iptable(ip1, ip2) VALUES(INET_ATON('10.198.1.1'), '10.198.1.1');
  INSERT INTO t_iptable(ip1, ip2) VALUES(INET_ATON('255.255.255.255'), '255.255.255.255');
  
  -- 测试结果
  -- INET_ATON()	Return the numeric value of an IP address		
  -- INET_NTOA()	Return the IP address from a numeric value		
  -- INET6_ATON()	Return the numeric value of an IPv6 address		
  -- INET6_NTOA()	Return the IPv6 address from a numeric value
  SELECT INET_NTOA(ip1), ip2 FROM t_iptable;
  +-----------------+-----------------+
  | INET_NTOA(ip1)  | ip2             |
  +-----------------+-----------------+
  | 0.0.0.0         | 0.0.0.0         |
  | 10.198.1.1      | 10.198.1.1      |
  | 255.255.255.255 | 255.255.255.255 |
  +-----------------+-----------------+
  ```

* 散列值：使用定长二进制类型（`binary`），而不是按十六进制标记法使用字符串类型（`char`）来存储散列值：https://stackoverflow.com/questions/614476/storing-sha1-hash-values-in-mysql

  ```sql
  -- 测试表
  CREATE TABLE `t_hash` (
    `hash1` binary(20) NOT NULL COMMENT '散列值',
    `hash2` char(40) NOT NULL COMMENT '散列值'
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  
  -- 测试数据
  INSERT INTO t_hash(hash1, hash2) VALUES(0xE562F69EC36E625116376F376D991E41613E9BF3, 'E562F69EC36E625116376F376D991E41613E9BF3');
  INSERT INTO t_hash(hash1, hash2) VALUES(X'E562F69EC36E625116376F376D991E41613E9BF3', 'E562F69EC36E625116376F376D991E41613E9BF3');
  INSERT INTO t_hash(hash1, hash2) VALUES(UNHEX('E562F69EC36E625116376F376D991E41613E9BF3'), 'E562F69EC36E625116376F376D991E41613E9BF3');
  
  -- 测试结果
  -- BIT_LENGTH()	Return length of a string in bits
  -- LENGTH()	Return length of a string in bytes
  SELECT HEX(hash1), BIT_LENGTH(hash1), LENGTH(hash1), hash2, BIT_LENGTH(hash2), LENGTH(hash2) FROM t_hash;
  +------------------------------------------+-------------------+---------------+------------------------------------------+-------------------+---------------+
  | HEX(hash1)                               | BIT_LENGTH(hash1) | LENGTH(hash1) | hash2                                    | BIT_LENGTH(hash2) | LENGTH(hash2) |
  +------------------------------------------+-------------------+---------------+------------------------------------------+-------------------+---------------+
  | E562F69EC36E625116376F376D991E41613E9BF3 |               160 |            20 | E562F69EC36E625116376F376D991E41613E9BF3 |               320 |            40 |
  +------------------------------------------+-------------------+---------------+------------------------------------------+-------------------+---------------+
  ```

## Avoid `NULL` if possible

避免使用 `NULL`：

> A lot of tables include nullable columns even when the application does not need to store `NULL` (the absence of a value), merely because it’s the default. It’s usually best to specify columns as `NOT NULL` unless you intend to store `NULL` in them.
>
> It’s harder for MySQL to optimize queries that refer to nullable columns, because they make indexes, index statistics, and value comparisons more complicated. A nullable column uses more storage space and requires special processing inside MySQL. When a nullable column is indexed, it requires an extra byte per entry and can even cause a fixed-size index (such as an index on a single integer column) to be converted to a variable-sized one in MyISAM.
>
> The performance improvement from changing `NULL` columns to `NOT NULL` is usually small, so don’t make it a priority to find and change them on an existing schema unless you know they are causing problems. However, if you’re planning to index columns, avoid making them nullable if possible.
>
> There are exceptions, of course. For example, it’s worth mentioning that InnoDB stores `NULL` with a single bit, so it can be pretty space-efficient for sparsely populated data. This doesn’t apply to MyISAM, though.

`NULL` 列使得 MySQL 索引、索引统计和值比较都更复杂。值可为 `NULL` 的列会使用更多的存储空间（例如当可为 `NULL` 的列被索引时，每个索引记录需要一个额外的字节），在 MySQL 里也需要特殊处理。如果计划在列上建索引，应该尽量避免。

**你应该用 0、一个特殊的值或者一个空串代替 NULL值。** 

参考：《[一千个不用 Null 的理由](https://my.oschina.net/leejun2005/blog/1342985)》

#  常用数据类型

## 数字类型

### 比特类型

`BIT[(M)]` 比特类型，*M* 为 1~64 bit(s)。

`b'value'` 符号可用于指定比特值。`value` 是一组使用 0 和 1 编写的二进制值。例如 `b'111'` 和 `b'10000000'` 分别代表 `7` 和 `128` 。详见《[Bit-Value Literals](https://dev.mysql.com/doc/refman/5.7/en/bit-value-literals.html)》。

如果赋值给小于 *M* 位长的 `BIT(M)` 类型列，则该值左侧用零填充。例如，为 `BIT(6)` 列赋值 `b'101'` 实际上等于赋值 `b'000101'`。

`BIT(1)` 常用来表示**布尔类型**，`b'0'` 表示 `false`，`b'1'` 表示 `true`。

1 byte = 8 bit。

### 整数类型

| Data Type   | Storage Required | Data Range（`signed`） | Data Range（`unsigned`） | Description                                                  |
| ----------- | ---------------- | ---------------------- | ------------------------ | ------------------------------------------------------------ |
| `TINYINT`   | 8 bits, 1 Byte   | -2^7 ~ 2^7-1           | 0 ~ 2^8-1                | 同义词 `BOOL`、`BOOLEAN` ，0 为 false，!0 为 true            |
| `SMALLINT`  | 16 bits, 2 Bytes | -2^15 ~ 2^15-1         | 0 ~ 2^16-1               |                                                              |
| `MEDIUMINT` | 24 bits, 3 Bytes | -2^23 ~ 2^23-1         | 0 ~ 2^24-1               |                                                              |
| `INT`       | 32 bits, 4 Bytes | -2^31 ~ 2^31-1         | 0 ~ 2^32-1               | 同义词 `INTEGER`                                             |
| `BIGINT`    | 64 bits, 8 Bytes | -2^63 ~ 2^63-1         | 0 ~ 2^64-1               | `SERIAL` 等于 `BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE` 的别名 ，适用于创建主键 |

`INT[(M)] [UNSIGNED] [ZEROFILL] [AUTO_INCREMENT]` 整数类型

* *`M`* 最大显示宽度，例如 `INT(11)` ：

  这个属性对于大多数应用都是没有意义的：它不会限制值的合法范围，只是规定了 MySQL 的一些交互工具用来**显示字符**的个数，最大值为 255，一般配合 `ZEROFILL` 使用。对于存储和计算来说，`INT(1)` 和 `INT(20)` 是相同的，并不会影响该类型的占用字节数。

* `ZEROFILL` 填充零：

  顾名思义就是用 “0” 填充的意思，也就是在数字位数不够（< *`M`* ）的空间用字符 “0” 填满。只在一些交互工具中有效，例如 MyCli。

- `UNSIGNED` 无符号：

  整数类型有可选的 `UNSIGNED` 属性，表示不允许负值，可以使正整数的上限提升一倍：

  |                  | 公式                 | 例子                                   |
  | ---------------- | -------------------- | -------------------------------------- |
  | 有符号的取值范围 | -2^(N-1) ~ 2^(N-1)-1 | `tinyint` -2^7 ~ 2^7-1 (-128 ~ 127)    |
  | 无符号的取值范围 | 0  ~ 2^N-1           | `tinyint UNSIGNED` 0 ~ 2^8-1 (0 ~ 255) |

  N 为存储长度的位数（1 Byte = 8 bits）。

* `AUTO_INCREMENT` 自动递增：

  在需要产生唯一标识符或顺序值时，可利用此属性，这个属性只用于整数类型。`AUTO_INCREMENT` 值一般从 1 开始，每行增加 1。 一个表中最多只能有一个 `AUTO_INCREMENT` 列 。对于任何想要使用 `AUTO_INCREMENT` 的列，应该定义为 `NOT NULL`，并定义为 `PRIMARY KEY` 或定义为 `UNIQUE` 键。

### 浮点类型（近似值）

`FLOAT[(M,D)] [UNSIGNED] [ZEROFILL]` 单精度浮点类型（floating-point number），*M* 是总位数，*D* 是小数点后面的位数。如果 *M* 和 *D* 省略，值将存储到硬件允许的限制。单精度浮点数精确到约 7 位小数。

`DOUBLE[(M,D)] [UNSIGNED] [ZEROFILL]` 双精度浮点类型（floating-point number），*M* 是总位数，*D* 是小数点后面的位数。如果 *M* 和 *D* 省略，值将存储到硬件允许的限制。双精度浮点数精确到小数点后 15 位。

| Data Type | Storage Required | Data Range                                                   |
| --------- | ---------------- | ------------------------------------------------------------ |
| `FLOAT`   | 32 bits, 4 Bytes | `-3.402823466E+38` to `-1.175494351E-38`, `0`, and `1.175494351E-38` to `3.402823466E+38` |
| `DOUBLE`  | 64 bits, 8 Bytes | `-1.7976931348623157E+308` to `-2.2250738585072014E-308`, `0`, and`2.2250738585072014E-308` to `1.7976931348623157E+308` |

因为浮点值是近似值而不是作为精确值存储的，比值时可能会导致问题。详见《[Problems with Floating-Point Values](https://dev.mysql.com/doc/refman/5.7/en/problems-with-float.html)》。

### 定点类型（精确值）

`DECIMAL[(M[,D])] [UNSIGNED] [ZEROFILL]` 定点类型（fixed-point number），用于存储高精度数值，如货币数据。

*M* 是总位数（精度，precision），*D* 是小数点后的位数（标度，scale）。小数点和（对于负数） `-` 符号不计入 *M*。如果 *D* 为 0，则值不包含小数点或小数部分。如果指定 `UNSIGNED`，则不允许负值。`DECIMAL` 的所有基本运算 (+, -, *, /) 都以 65 位数的最大精度完成。

| Data Type | *M* 精度范围（总位数） | *D* 标度范围（小数位数） | 备注                             |
| --------- | ---------------------- | ------------------------ | -------------------------------- |
| `DECIMAL` | 0~65，默认 10          | 0~30，默认 0             | 同义词 `DEC`、`NUMERIC`、`FIXED` |

例如 ：

```mysql
salary DECIMAL(5,2)
```

可以存储在 *salary* 列中的值范围从 -999.99 ~ 999.99。

`DECIMAL` 以二进制格式存储值，每 4 个字节存 9 个数字。例如，`DECIMAL(18,9)` 小数点两边各存储 9 个数字，一共使用 9 个字节：小数点前的数字使用 4 个字节，小数点后的数字使用 4 个字节，小数点本身占 1 个字节。详见《[Precision Math](https://dev.mysql.com/doc/refman/5.7/en/precision-math.html)》。

因为需要额外的空间和计算开销，所以应该尽量只在对小数进行精确计算时才使用 `DECIMAL` —— 例如存储财务数据。但在数据量比较大的时候，可以考虑使用 `BIGINT` 代替 `DECIMAL` ，将需要存储的货币单位根据小数的位数乘以相应的倍数即可。假设要存储的财务数据精确到万分之一分，则可以把所有金额乘以一百万，然后将结果存储在 `BIGINT` 里，这样可以同时避免浮点存储计算不精确和 `DECIMAL` 精确计算代价高的问题。

## 字符串类型

In the following table

* *`M`* represents the declared column length in 
  * **bytes** for binary string types (`BINARY(M)`、`VARBINARY(M)`)
  * **characters** for nonbinary string types (`CHAR(M)`、`VARCHAR(M)`)
* *`L`* represents the actual length in **bytes** of a given string value.

|  | Binary Strings (Byte Strings) | Nonbinary Strings (Character Strings) | Storage Required                                             |
| ------------ | ------------------------------------------------------------ | ------------ | ------------ |
| **Fixed-length types** |     | `CHAR(M)` | *`L`* = *`M`* × *`w`* bytes, 0 < *`M`* <= 255, where *`w`* is the number of bytes required for the maximum-length character in the character set. |
| **Fixed-length types** | `BINARY(M)` |  | *`M`* bytes, 0 <= *`M`* <= 255 |
| **Variable-length types** | `VARBINARY(M)` | `VARCHAR(M)` | *`L`* = *`M`* × *`w`* bytes + 1 bytes if column values require 0 − 255 bytes<br>*`L`* = *`M`* × *`w`* bytes + 2 bytes if values may require more than 255 bytes<br/>其有效最大字节长度取决于**行大小限制**（默认 65,535 Bytes，在所有列中共享） ，参考：《[表列数量和行数限制](https://dev.mysql.com/doc/refman/5.7/en/column-count-limit.html)》。 |
| **Variable-length types** | `TINYBLOB`     | `TINYTEXT` | *`L`* + 1 bytes, where *`L`* < 2^8 = 256 bytes |
| **Variable-length types** | `BLOB`             | `TEXT`       | *`L`* + 2 bytes, where *`L`* < 2^16 = 64 KB    |
| **Variable-length types** | `MEDIUMBLOB` | `MEDIUMTEXT` | *`L`* + 3 bytes, where *`L`* < 2^24 = 16 MB    |
| **Variable-length types** | `LONGBLOB`     | `LONGTEXT` | *`L`* + 4 bytes, where *`L`* < 2^32 = 4 GB     |

variable-length types 变长类型需要额外的 1 到 4 个字节记录长度：

* 1 Byte = 8 bits 刚好可以记录 0~2^8-1 (255) bytes
* 2 Bytes = 16 bits 刚好可以记录 0~2^16-1 (65,535) bytes
* 3 Bytes = 24 bits 刚好可以记录 0~2^24 bytes
* 4 Bytes = 32 bits 刚好可以记录 0~2^32 bytes

|                                           | Description                                                  |      |
| ----------------------------------------- | ------------------------------------------------------------ | ---- |
| **Binary Strings (Byte Strings)**         | They have the `binary` character set and collation, and comparison and sorting are based on the numeric values of the bytes in the values. |      |
| **Nonbinary Strings (Character Strings)** | They have a [character set](https://dev.mysql.com/doc/refman/5.7/en/charset.html) other than `binary`, and values are sorted and compared based on the collation of the character set. |      |

对于 Nonbinary Strings (Character Strings)，*M* 和 *L* 换算关系如下：

| 字符集（Character Sets） | *M*         | *L*     |
| ------------------------ | ----------- | ------- |
| `latin1`                 | 1 character | 1 byte  |
| `gbk`                    | 1 character | 2 bytes |
| `utf8`                   | 1 character | 3 bytes |
| `utf8mb4`                | 1 character | 4 bytes |

### `BINARY` 和 `VARBINARY` 类型

https://dev.mysql.com/doc/refman/5.7/en/binary-varbinary.html

> The `BINARY` and `VARBINARY` types are similar to [`CHAR`](https://dev.mysql.com/doc/refman/5.7/en/char.html) and [`VARCHAR`](https://dev.mysql.com/doc/refman/5.7/en/char.html), except that they store binary strings rather than nonbinary strings. That is, they store byte strings rather than character strings. This means they have the `binary` character set and collation, and comparison and sorting are based on the numeric values of the bytes in the values.

#### Hexadecimal Literals

https://en.wikipedia.org/wiki/Octal

https://dev.mysql.com/doc/refman/8.0/en/hexadecimal-literals.html

Hexadecimal literal values are written using `X'val'` or `0xval` notation, where *`val`* contains hexadecimal digits (`0..9`, `A..F`). Lettercase of the digits and of any leading `X` does not matter. A leading `0x` is case-sensitive and cannot be written as `0X`.

Legal hexadecimal literals:

```sql
X'01AF'
X'01af'
x'01AF'
x'01af'
0x01AF
0x01af
```

By default, a hexadecimal literal is a **binary string**, where **each pair of hexadecimal digits** represents a character:

![ASCII](/img/mysql/charset/ascii.png)

```SQL
-- BIT_LENGTH()	Return length of a string in bits
-- LENGTH()	Return length of a string in bytes
-- CHARSET()	Returns the character set of the string argument.
SELECT 0x41, X'41', UNHEX('41'), BIT_LENGTH(0x41), LENGTH(0x41), CHARSET(0x41);
+------+-------+-------------+------------------+--------------+---------------+
| 0x41 | X'41' | UNHEX('41') | BIT_LENGTH(0x41) | LENGTH(0x41) | CHARSET(0x41) |
+------+-------+-------------+------------------+--------------+---------------+
| A    | A     | A           |                8 |            1 | binary        |
+------+-------+-------------+------------------+--------------+---------------+
```

> [`UNHEX(str)`](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_unhex)
>
> For a string argument *`str`*, [`UNHEX(str)`](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_unhex) interprets **each pair of characters in the argument as a hexadecimal number** and converts it to the byte represented by the number. The return value is a **binary string**.
>

For information about introducers, see [Section 10.3.8, “Character Set Introducers”](https://dev.mysql.com/doc/refman/8.0/en/charset-introducer.html).

### `CHAR` 和 `VARCHAR` 类型

https://dev.mysql.com/doc/refman/5.7/en/char.html

`CHAR` 和 `VARCHAR` 这两种类型很相似，但它们被存储和检索的方式不同。区别如下：

| Data Type    | 尾部空格是否保留 | 描述                                                         | 适用情况                                                     |
| ------------ | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `CHAR(M)`    | 是               | 用于存储定长字符串。字符长度不足时会填充尾部空格到指定的长度。 | 存储很短的字符串，或者所有值都接近同一个长度。或经常变更的数据，定长的 `CHAR` 类型不容易产生碎片。 |
| `VARCHAR(M)` | 否               | 用于存储可变长字符串，是最常见的字符串数据类型。它比定长类型更节省空间，因为它仅使用必要的空间。 | 字符的最大字节长度比平均长度大很多；列的更新很少，所以碎片不是问题；使用了像 UTF-8 这样复杂的字符集，每个字符都使用不同的字节数进行存储。 |

例子：下表通过存储各种字符串值到 `CHAR(4)` 和 `VARCHAR(4)` 列展示 `CHAR` 和 `VARCHAR` 之间的差别（假设该列使用单字节字符集，例如 `latin1`）：

| 值           | `CHAR(4)`           | 实际字节长度 | `VARCHAR(4)` | 实际字节长度 |
| ------------ | ------------------- | ------------ | ------------ | ------------ |
| `''`         | `'    '` (四个空格) | 4 bytes      | `''`         | 1 byte       |
| `'ab'`       | `'ab  '` (两个空格) | 4 bytes      | `'ab'`       | 3 bytes      |
| `'abcd'`     | `'abcd'`            | 4 bytes      | `'abcd'`     | 5 bytes      |
| `'abcdefgh'` | `'abcd'`            | 4 bytes      | `'abcd'`     | 5 bytes      |

### `BLOB` 和 `TEXT` 类型

https://dev.mysql.com/doc/refman/5.7/en/blob.html

与其它类型不同，MySQL 把每个 `BLOB` 和 `TEXT` 值当做一个独立的对象处理。存储引擎在存储时通常会做特殊处理。当 `BLOB` 和 `TEXT` 值太大时，InnoDB 会使用专门的“外部”存储区域来进行存储，**此时每个值在行内需要 1~4 个字节存储一个指针，然后在外部存储区域存储实际的值**。

MySQL 不能将 `BLOB` 和 `TEXT` 列全部长度的字符串进行索引，也不能使用这些索引消除排序，因此可以使用“前缀索引”解决这个问题。

## 日期与时间类型

| Data Type | **Storage Required before MySQL 5.6.4** | **Storage Required as of MySQL 5.6.4** | 0 值      | 取值范围                                          |
| ----------- | -------- | ------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| [`YEAR`](https://dev.mysql.com/doc/refman/5.7/en/year.html) | 1 byte, little endian | Unchanged                                        | `0000`              | `1901` to `2155`                               |
| [`DATE`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) | 3 bytes, little endian | Unchanged                                        | `0000-00-00`        | `1000-01-01` to `9999-12-31` |
| [`TIME[(fsp)]`](https://dev.mysql.com/doc/refman/5.7/en/time.html) | 3 bytes, little endian | 3 bytes + fractional-seconds storage, big endian | `00:00:00`          | `-838:59:59.000000` to `838:59:59.000000` |
| [`DATETIME[(fsp)]`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) | 8 bytes, little endian | 5 bytes + fractional-seconds storage, big endian | `0000-00-00 00:00:00` | `1000-01-01 00:00:00.000000` to `9999-12-31 23:59:59.999999` |
| [`TIMESTAMP[(fsp)]`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) | 4 bytes, little endian | 4 bytes + fractional-seconds storage, big endian | `0000-00-00 00:00:00` UTC | `1970-01-01 00:00:01.000000` UTC to `2038-01-19 03:14:07.999999` UTC |

### TIMESTAMP[(fsp)]

[Section 11.2.1, "Date and Time Data Type Syntax"](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-type-syntax.html)

> A timestamp. The range is `'1970-01-01 00:00:01.000000'` UTC to `'2038-01-19 03:14:07.999999'` UTC. [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) values are stored as the number of seconds since the epoch (`'1970-01-01 00:00:00'` UTC). A [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) cannot represent the value `'1970-01-01 00:00:00'` because that is equivalent to 0 seconds from the epoch and the value 0 is reserved for representing `'0000-00-00 00:00:00'`, the “zero” [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) value.
>
> An optional *`fsp`* value in the range from 0 to 6 may be given to specify fractional seconds precision. A value of 0 signifies that there is no fractional part. If omitted, the default precision is 0.
>
> ...

`TIMESTAMP` 类型的范围如下：

| 时间戳             | 二进制字面量                        | 时间                      |
| ------------------ | ----------------------------------- | ------------------------- |
| 0                  | 00000000 00000000 00000000 00000000 | `0000-00-00 00:00:00` UTC |
| 1                  | 00000000 00000000 00000000 00000001 | `1970-01-01 00:00:01` UTC |
| 2^31-1, 2147483647 | 01111111 11111111 11111111 11111111 | `2038-01-19 03:14:07` UTC |

`TIMESTAMP` 类型的时区处理：

> MySQL converts `TIMESTAMP` values from the current time zone to UTC for storage, and back from UTC to the current time zone for retrieval. (This does not occur for other types such as `DATETIME`.) By default, the current time zone for each connection is the server's time. The time zone can be set on a per-connection basis. As long as the time zone setting remains constant, you get back the same value you store. If you store a `TIMESTAMP` value, and then change the time zone and retrieve the value, the retrieved value is different from the value you stored. This occurs because the same time zone was not used for conversion in both directions. The current time zone is available as the value of the [`time_zone`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_time_zone) system variable. For more information, see [Section 5.1.13, “MySQL Server Time Zone Support”](https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html).

用无符号 `int` 或 `bigint` 存储时间戳也是一种解决方案，两种方案对比如下：

|                            | `TIMESTAMP`                                                  | `INT`、`BIGINT`                                              |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 时间范围                   | 存在 [2K38 问题](/posts/unix-timestamp/#Y2K38-Year-2038-problem) | 时间范围更大                                                 |
| 时区支持                   | 无时区，便于国际化业务                                       | 无时区，便于国际化业务                                       |
| 自动初始化和更新           | 支持                                                         | 不支持                                                       |
| 是否支持使用时间戳整数查询 | 不支持                                                       | 支持                                                         |
| DBMS 查询显示              | 支持用本地时区显示（缺省情况下，每个连接使用服务器时区。也可以为每个连接设置时区） | 需通过 [`FROM_UNIXTIME(unix_timestamp[,format])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_from-unixtime) 函数转换，否则阅读困难 |

### DATETIME[(fsp)]

[Section 11.2.1, "Date and Time Data Type Syntax"](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-type-syntax.html)

> A `date` and `time` combination. The supported range is `'1000-01-01 00:00:00.000000'` to `'9999-12-31 23:59:59.999999'`. MySQL displays [`DATETIME`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) values in `YYYY-MM-DD hh:mm:ss[.fraction]` format, but permits assignment of values to [`DATETIME`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) columns using either strings or numbers.
>
> An optional *`fsp`* value in the range from 0 to 6 may be given to specify fractional seconds precision. A value of 0 signifies that there is no fractional part. If omitted, the default precision is 0.

`DATETIME` 类型是一个本地时间，**与时区无关**。

默认情况下，MySQL 以一种可排序的、无歧义的格式显示 `DATETIME` 值，例如“2018-01-16 22:37:08”。这是 ANSI 标准定义的日期和时间显示方法。

`DATETIME` 类型允许使用字符串类型或整数类型进行**赋值**：

```mysql
-- TODO
```

当 `DATETIME` 类型与整型比较时，`DATETIME` 类型会自动转为整型。利用这个特性可以方便快速比较，例如查询时间范围为 2018-02-15 00:00:00 到 2018-02-16 00:00:00：

```mysql
select count(1) 
from t_table 
where createTime between 20180215 and 20180216;
```

`DATETIME` 类型非小数部分的编码如下。参考：[Section 10.9, "Date and Time Data Type Representation"](https://dev.mysql.com/doc/internals/en/date-and-time-data-type-representation.html)

```
 1 bit  sign           (1= non-negative, 0= negative)
17 bits year*13+month  (year 0-9999, month 0-12)
 5 bits day            (0-31)
 5 bits hour           (0-23)
 6 bits minute         (0-59)
 6 bits second         (0-59)
---------------------------
40 bits = 5 bytes
```

### 存储精度（小数秒）

需要注意的是，MySQL 升级到 5.6 之后对日期与时间类型做过调整，可以精确到微秒并指定其精度（最多 6 位），参考 [Changes in MySQL 5.6](https://dev.mysql.com/doc/refman/5.6/en/upgrading-from-previous-series.html)：

![incompatible_change_of_date_and_time_type](/img/mysql/datatype/incompatible_change_of_date_and_time_type.png)

参考 [Date and Time Type Storage Requirements](https://dev.mysql.com/doc/refman/5.6/en/storage-requirements.html#data-types-storage-reqs-date-time) 下表列明了日期与时间类型在 MySQL 5.6.4 前后的变化：

![date_and_time_type_storage_requirements](/img/mysql/datatype/date_and_time_type_storage_requirements.png)

通过分析精确到小数部分的秒（Fractional Seconds Precision）所支持的最大十进制数值，并将其转换为二进制表示，可知为什么精度越高所需的存储空间越多：

| Fractional Seconds Precision | Maximum Decimal Representation | Maximum Binary Representation           | Storage Required |
| ---------------------------- | ------------------------------ | --------------------------------------- | ---------------- |
| 0                            | 0                              | 0 (0 bit)                               | 0 byte           |
| 1, 2                         | 99                             | 0110 0011 (8 bits)                      | 1 byte           |
| 3, 4                         | 9,999                          | 0010 0111 0000 1111 (16 bits)           | 2 bytes          |
| 5, 6                         | 999,999                        | 0000 1111 0100 0010 0011 1111 (24 bits) | 3 bytes          |

有关于时间值的内部表示的详细信息，参考 [MySQL Internals: Important Algorithms and Structures - Date and Time Data Type Representation](https://dev.mysql.com/doc/internals/en/date-and-time-data-type-representation.html)

### 最佳实践

* MySQL 有多种表示日期的数据类型，比如，当只记录年信息的时候，可以使用 `YEAR` 类型，而没有必要使用 `DATE` 类型。

* 节省存储空间，仅在必要时为 [`TIME`](https://dev.mysql.com/doc/refman/5.7/en/time.html), [`DATETIME`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html), and [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) 指定精度：

  > A `DATETIME` or `TIMESTAMP` value can include a trailing fractional seconds part in up to microseconds (6 digits) precision. In particular, any fractional part in a value inserted into a `DATETIME` or `TIMESTAMP` column is stored rather than discarded. With the fractional part included, the format for these values is `'YYYY-MM-DD hh:mm:ss[.fraction]'`, the range for `DATETIME` values is `'1000-01-01 00:00:00.000000'` to `'9999-12-31 23:59:59.999999'`, and the range for `TIMESTAMP` values is `'1970-01-01 00:00:01.000000'` to `'2038-01-19 03:14:07.999999'`. The fractional part should always be separated from the rest of the time by a decimal point; no other fractional seconds delimiter is recognized. For information about fractional seconds support in MySQL, see [Section 11.2.7, “Fractional Seconds in Time Values”](https://dev.mysql.com/doc/refman/5.7/en/fractional-seconds.html).

* 利用自动初始化和更新功能，为 `create_time`、`update_time` 字段赋值：

  > The `TIMESTAMP` and `DATETIME` data types offer automatic initialization and updating to the current date and time. For more information, see [Section 11.2.6, “Automatic Initialization and Updating for TIMESTAMP and DATETIME”](https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html).

  ```SQL
  CREATE TABLE t1 (
    ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    dt DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  );
  ```

* 注意每一个类型都有合法的取值范围，当指定确实不合法的值时系统将 "零" 值插入到数据库中。但要注意开启相关 [SQL mode](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_sql_mode)：

  > Invalid `DATE`, `DATETIME`, or `TIMESTAMP` values are converted to the “zero” value of the appropriate type (`'0000-00-00'` or `'0000-00-00 00:00:00'`), if the SQL mode permits this conversion. The precise behavior depends on which if any of strict SQL mode and the [`NO_ZERO_DATE`](https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_no_zero_date) SQL mode are enabled; see [Section 5.1.10, “Server SQL Modes”](https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html).

# 默认值

* 默认值必须是常量，而不能是一个函数或表达式。举个栗子，这意味着你不能将日期列的默认值设置为诸如 `NOW()` 或 `CURRENT_DATE` 之类的函数的值。唯一例外是你可以指定 `CURRENT_TIMESTAMP` 为 `TIMESTAMP` 和 `DATETIME` 列的默认值。
* 隐式默认值定义如下：
  * 数字类型
    * 对于使用 `AUTO_INCREMENT` 属性声明的整数类型或浮点类型，默认值为下一个序列值。
    * 否则默认值为 `0` 。
  * 字符串类型
    * `ENUM` 的默认值为第一个枚举值。
    * `BLOB` 、`TEXT` 列无法指定默认值。
    * 其它类型的默认值为空字符串。
  * 日期与时间类型
    * `TIMESTAMP`
      * 如果系统变量 `explicit_defaults_for_timestamp` 开启，其默认值为 0 值。
      * 否则表中第一列 `TIMESTAMP` 的默认值为当前时间。
    * 其它类型的默认值为相应的 0 值。

# Java, JDBC, and MySQL Types

参考：

[JDBC SQL 和 Java 数据类型映射总结](/posts/java-jdbc-mapping-sql-and-java-types/)

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-type-conversions.html

![Java, JDBC, and MySQL Types](/img/java/jdbc/mysql-types.png)

# 参考

https://dev.mysql.com/doc/refman/5.7/en/data-types.html

https://dev.mysql.com/doc/refman/5.7/en/column-count-limit.html

《[MySQL 数据类型：UNSIGNED 注意事项](https://www.cnblogs.com/blankqdb/archive/2012/11/03/blank_qdb.html)》

《[MySQL 数据类型：二进制类型](https://blog.csdn.net/u011794238/article/details/50962702)》

《[MySQL 5.6 时间数据类型功能获得改进](http://tech.it168.com/a2013/1013/1544/000001544067.shtml)》

《[一只天价股票把纳斯达克系统搞“崩了”!](https://mp.weixin.qq.com/s/Zc2X-K7SCXEJq_EhgAI8ig)》

《[如果要存ip地址，用什么数据类型比较好？](https://mp.weixin.qq.com/s/n2vEImxO_AfE28H9dETw4A)》