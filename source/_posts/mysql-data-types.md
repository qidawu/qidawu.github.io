---
title: MySQL 常用数据类型
date: 2018-01-05 22:27:21
updated:
tags: MySQL
typora-root-url: ..
---

# 原则

MySQL 支持的数据类型非常多，选择正确的数据类型对于获得高性能至关重要。不管存储哪种类型的数据，下面几个简单的原则都有助于作出更好的选择。

## 更小的通常更好

更小的数据类型通常更快，因为占用更少的磁盘、内存和 CPU 缓存，并且处理时需要的 CPU 周期也更少。

## 简单就好

简单数据类型的操作通常需要更少的 CPU 周期、索引性能更好。例如，整型比字符操作代价更低，因为字符集和校对规则（排序规则）使得字符比整型更复杂。

有几个例子：

* 使用日期与时间类型，而不是字符串来存储日期和时间，以便排序和格式转换。

* 使用整型，而不是字符串来存储 IP 地址。MySQL 提供了两个函数来处理 IP 地址：

  ```sql
  -- 使用 INT unsigned 类型存储（存储范围：0 ~ 2^32-1）
  SELECT inet_aton('0.0.0.0');  --结果：0
  SELECT inet_aton('192.168.1.1');  --结果：3232235777
  SELECT inet_aton('255.255.255.255');  --结果：4294967295
  
  SELECT inet_ntoa('3232235777');  --结果：192.168.1.1
  ```

* 使用定长二进制类型（如 `binary`），而不是字符串来存储散列值：

  * `MD5` 128 bit / 4 = 32 length
  * `SHA-1` 160 bit / 4 = 40 length
  * `SHA-224` 224 bit / 4 = 56 length
  * `SHA-256` 256 bit / 4 = 64 length
  * `SHA-384` 384 bit / 4 = 96 length
  * `SHA-512` 512 bit / 4 = 128 length

  ```sql
  -- 示例表
  CREATE TABLE `t_hash` (
    `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键 ID',
    `hash` varbinary(40) NOT NULL COMMENT '散列值',
    PRIMARY KEY (`id`),
    KEY `idx_hash` (`hash`) USING BTREE
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  
  -- 测试数据
  INSERT INTO t_hash(hash) values(md5('0.0.0.0'));
  INSERT INTO t_hash(hash) values(sha1('0.0.0.0'));
  INSERT INTO t_hash(hash) values('你好吗');
  
  -- 测试结果
  -- length() 函数返回值的单位为字节。
  -- 由于二进制类型无字符集，因此中文乱码
  select hash, length(hash), char_length(hash) from t_hash order by id;
  +------------------------------------------+--------------+-------------------+
  | hash                                     | length(hash) | char_length(hash) |
  +------------------------------------------+--------------+-------------------+
  | f1f17934834ae2613699701054ef9684         |           32 |                32 |
  | e562f69ec36e625116376f376d991e41613e9bf3 |           40 |                40 |
  | 浣犲ソ鍚?                                 |            9 |                 9 |
  +------------------------------------------+--------------+-------------------+
  ```

## 避免使用 NULL

`NULL` 列使得 MySQL 索引、索引统计和值比较都更复杂。可为 `NULL` 的列会使用更多的存储空间（例如当可为 `NULL` 的列被索引时，每个索引记录需要一个额外的字节），在 MySQL 里也需要特殊处理。如果计划在列上建索引，应该尽量避免。

#  常用数据类型

## 数字类型

### 比特类型

`BIT[(M)]` 比特类型，*M* 为 1~64 位比特。

`b'value'` 符号可用于指定比特值。`value` 是一组使用 0 和 1 编写的二进制值。例如 `b'111'` 和 `b'10000000'` 分别代表 `7` 和 `128` 。详见《[Bit-Value Literals](https://dev.mysql.com/doc/refman/5.7/en/bit-value-literals.html)》。

如果赋值给小于 *M* 位长的 `BIT(M)` 类型列，则该值左侧用零填充。例如，为 `BIT(6)` 列赋值 `b'101'` 实际上等于赋值 `b'000101'`。

`BIT(1)` 常用来表示布尔类型，`b'0'` 表示 `false`，`b'1'` 表示 `true`。

1 byte = 8 bit。

### 整数类型（精确值）

MySQL 可以使用以下几种整数类型，存储的值的范围从“-2^(N-1)”到“2^(N-1)-1”，其中 N 是存储空间的位数（1 byte = 8 bit）：

| 类型        | 字节长度 | 取值范围（有符号） | 取值范围（无符号） | 备注                                                         |
| ----------- | -------- | ------------------ | ------------------ | ------------------------------------------------------------ |
| `TINYINT`   | 1 byte   | -2^7 ~2^7-1        | 0~2^8-1            | 同义词 `BOOL`、`BOOLEAN` ，0 为 false，!0 为 true            |
| `SMALLINT`  | 2 bytes  | -2^15 ~2^15-1      | 0~2^16-1           |                                                              |
| `MEDIUMINT` | 3 bytes  | -2^23 ~2^23-1      | 0~2^24-1           |                                                              |
| `INT`       | 4 bytes  | -2^31 ~2^31-1      | 0~2^32-1           | 同义词 `INTEGER`                                             |
| `BIGINT`    | 8 bytes  | -2^63 ~2^63-1      | 0~2^64-1           | `SERIAL` 等于 `BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE` 的别名 ，适用于创建主键 |

`INT[(M)] [UNSIGNED] [ZEROFILL] [AUTO_INCREMENT]` 整数类型

* *`M`* 最大显示宽度，例如 `INT(11)` ：

  这个属性对于大多数应用都是没有意义的：它不会限制值的合法范围，只是规定了 MySQL 的一些交互工具用来**显示字符**的个数，最大值为 255，一般配合 `ZEROFILL` 使用。对于存储和计算来说，`INT(1)` 和 `INT(20)` 是相同的，并不会影响该类型的占用字节数。

* `ZEROFILL` 填充零：

  顾名思义就是用 “0” 填充的意思，也就是在数字位数不够（< *`M`*）的空间用字符 “0” 填满。只在一些交互工具中有效，例如 MyCli。

- `UNSIGNED` 无符号：

  整数类型有可选的 `UNSIGNED` 属性，表示不允许负值，这大致可以使正数的上限提高一倍。例如 `tinyint UNSIGNED` 可以存储的范围是 0~255，而 `tinyint` 的存储范围是 -128~127。

* `AUTO_INCREMENT` 自动递增：

  在需要产生唯一标识符或顺序值时，可利用此属性，这个属性只用于整数类型。`AUTO_INCREMENT` 值一般从 1 开始，每行增加 1。 一个表中最多只能有一个 `AUTO_INCREMENT` 列 。对于任何想要使用 `AUTO_INCREMENT` 的列，应该定义为 `NOT NULL`，并定义为 `PRIMARY KEY` 或定义为 `UNIQUE` 键。

### 定点类型（精确值）

`DECIMAL[(M[,D])] [UNSIGNED] [ZEROFILL]` 定点类型（fixed-point number），用于存储高精度数值，如货币数据。

*M* 是总位数（精度，precision），*D* 是小数点后的位数（标度，scale）。小数点和（对于负数） `-` 符号不计入 *M*。如果 *D* 为 0，则值不包含小数点或小数部分。如果指定 `UNSIGNED`，则不允许负值。`DECIMAL` 的所有基本运算 (+, -, *, /) 都以 65 位数的最大精度完成。

| 类型        | *M* 精度范围（总位数） | *D* 标度范围（小数位数） | 备注                          |
| --------- | ------------- | -------------- | --------------------------- |
| `DECIMAL` | 0~65，默认 10    | 0~30，默认 0      | 同义词 `DEC`、`NUMERIC`、`FIXED` |

例如 ：

```mysql
salary DECIMAL(5,2)
```

可以存储在 *salary* 列中的值范围从 -999.99 ~ 999.99。

`DECIMAL` 以二进制格式存储值，每 4 个字节存 9 个数字。例如，`DECIMAL(18,9)` 小数点两边各存储 9 个数字，一共使用 9 个字节：小数点前的数字使用 4 个字节，小数点后的数字使用 4 个字节，小数点本身占 1 个字节。详见《[Precision Math](https://dev.mysql.com/doc/refman/5.7/en/precision-math.html)》。

因为需要额外的空间和计算开销，所以应该尽量只在对小数进行精确计算时才使用 `DECIMAL` —— 例如存储财务数据。但在数据量比较大的时候，可以考虑使用 `BIGINT` 代替 `DECIMAL` ，将需要存储的货币单位根据小数的位数乘以相应的倍数即可。假设要存储的财务数据精确到万分之一分，则可以把所有金额乘以一百万，然后将结果存储在 `BIGINT` 里，这样可以同时避免浮点存储计算不精确和 `DECIMAL` 精确计算代价高的问题。

### 浮点类型（近似值）

`FLOAT[(M,D)] [UNSIGNED] [ZEROFILL]` 单精度浮点类型（floating-point number），*M* 是总位数，*D* 是小数点后面的位数。如果 *M* 和 *D* 省略，值将存储到硬件允许的限制。单精度浮点数精确到约 7 位小数。

`DOUBLE[(M,D)] [UNSIGNED] [ZEROFILL]` 双精度浮点类型（floating-point number），*M* 是总位数，*D* 是小数点后面的位数。如果 *M* 和 *D* 省略，值将存储到硬件允许的限制。双精度浮点数精确到小数点后 15 位。

| 类型     | 字节长度 | 取值范围                                                     |
| -------- | -------- | ------------------------------------------------------------ |
| `FLOAT`  | 4 bytes  | `-3.402823466E+38` to `-1.175494351E-38`, `0`, and `1.175494351E-38` to `3.402823466E+38` |
| `DOUBLE` | 8 bytes  | `-1.7976931348623157E+308` to `-2.2250738585072014E-308`, `0`, and`2.2250738585072014E-308` to `1.7976931348623157E+308` |

因为浮点值是近似值而不是作为精确值存储的，比值时可能会导致问题。详见《[Problems with Floating-Point Values](https://dev.mysql.com/doc/refman/5.7/en/problems-with-float.html)》。

## 字符串类型

### CHAR 和 VARCHAR 类型

下列表格中，*M* 表示字符长度，*L* 表示实际字节存储长度。这两种类型很相似，但它们被存储和检索的方式不同。它们的最大字节长度 *L* 和尾部空格是否保留也不同。

| 类型         | 最大字节长度          | 尾部空格是否保留 | 描述                                                         | 适用情况                                                     |
| ------------ | --------------------- | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `VARCHAR(M)` | 65,535 bytes (2^16-1) | 否               | 用于存储可变长字符串，是最常见的字符串数据类型。它比定长类型更节省空间，因为它仅使用必要的空间。其有效最大长度取决于**行大小限制**（默认 65,535 字节，在所有列中共享） 和使用的字符集（例如， UTF-8 字符集每个字符最多可能需要三个字节，因此最多可以声明为 21,844 个字符）。参加《[表列数量和行数限制](https://dev.mysql.com/doc/refman/5.7/en/column-count-limit.html)》。 | 字符的最大长度比平均长度大很多；列的更新很少，所以碎片不是问题；使用了像 UTF-8 这样复杂的字符集，每个字符都使用不同的字节数进行存储。 |
| `CHAR(M)`    | 255 bytes (2^8-1)     | 是               | 用于存储定长字符串。长度不足时会填充尾部空格到指定的长度。   | 存储很短的字符串，或者所有值都接近同一个长度。例如，存储密码的 MD5 值；经常变更的数据，定长的 `CHAR` 类型不容易产生碎片。 |

不同于 `CHAR` 类型，`VARCHAR` 类型需要使用 1 或 2 个额外字节记录字符串的长度：如果列的最大长度小于或等于 255 字节，则只使用 1 个字节表示，否则使用 2 个字节：

> *L* + 1 bytes if column values require 0 − 255 bytes, *L* + 2 bytes if values may require more than 255 bytes

下表通过存储各种字符串值到 `CHAR(4)` 和 `VARCHAR(4)` 列说明了 `CHAR` 和 `VARCHAR` 之间的差别（假设该列使用单字节字符集，例如 latin1）：

| 值           | `CHAR(4)` | 需要存储 | `VARCHAR(4)` | 需要存储 |
| ------------ | --------- | -------- | ------------ | -------- |
| `''`         | `'    '`  | 4 bytes  | `''`         | 1 byte   |
| `'ab'`       | `'ab  '`  | 4 bytes  | `'ab'`       | 3 bytes  |
| `'abcd'`     | `'abcd'`  | 4 bytes  | `'abcd'`     | 5 bytes  |
| `'abcdefgh'` | `'abcd'`  | 4 bytes  | `'abcd'`     | 5 bytes  |

### BLOB 和 TEXT 类型

`TEXT` 表示二进制字符串（字节字符串），没有排序规则或字符集。`BLOB` 表示非二进制字符串（字符串），有排序规则、字符集。与其它类型不同，MySQL 把每个 `BLOB` 和 `TEXT` 值当做一个独立的对象处理。存储引擎在存储时通常会做特殊处理。当 `BLOB` 和 `TEXT` 值太大时，InnoDB 会使用专门的“外部”存储区域来进行存储，**此时每个值在行内需要 1~4 个字节存储一个指针，然后在外部存储区域存储实际的值**。它们的最大字节长度如下：

| 类型                       | 指针长度 | 实际值的最大字节长度              |
| -------------------------- | -------- | --------------------------------- |
| `TINYTEXT`、`TINYBLOB`     | 1 byte   | 256 bytes (2^8)                   |
| `TEXT`、`BLOB`             | 2 bytes  | 65,536 bytes (2^16)，64 KB        |
| `MEDIUMTEXT`、`MEDIUMBLOB` | 3 bytes  | 16,777,216 bytes (2^24)，16 MB    |
| `LONGTEXT`、`LONGBLOB`     | 4 bytes  | 4,294,967,296 bytes  (2^32)，4 GB |

MySQL 不能将 `BLOB` 和 `TEXT` 列全部长度的字符串进行索引，也不能使用这些索引消除排序，因此可以使用“前缀索引”解决这个问题。

## 日期与时间类型

* MySQL 有多种表示日期的数据类型，比如，当只记录年信息的时候，可以使用 `YEAR` 类型，而没有必要使用 `DATE` 类型。
* 每一个类型都有合法的取值范围，当指定确实不合法的值时系统将 "零" 值插入到数据库中。

| 类型        | 字节长度 | 默认值（0 值）      | 取值范围                                          | 备注                                                         |
| ----------- | -------- | ------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| `YEAR`      | 1 byte   | 0000                | 1901 ~ 2155                                       |                                                              |
| `DATE`      | 3 bytes  | 0000-00-00          | 1000-01-01 ~ 9999-12-31                           |                                                              |
| `TIME`      | 3 bytes  | 00:00:00            | -838:59:59 ~ 838:59:59                            |                                                              |
| `DATETIME`  | 5 bytes  | 0000-00-00 00:00:00 | 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59         | `DATETIME` 把日期和时间封装到格式为 `YYYYMMDDHHMMSS` 的整数中，与时区无关。默认情况下，MySQL 以一种可排序的、无歧义的格式显示 `DATETIME` 值，例如“2018-01-16 22:37:08”。这是 ANSI 标准定义的日期和时间显示方法。 |
| `TIMESTAMP` | 4 bytes  | 0000-00-00 00:00:00 | 1970-01-01 00:00:01 UTC ~ 2038-01-19 03:14:07 UTC | `TIMESTAMP` 显示的值依赖于时区。                             |

需要注意的是，MySQL 升级到 5.6 之后对日期与时间类型做过调整，可以精确到微秒并指定其精度（最多 6 位），参考 [Changes in MySQL 5.6](https://dev.mysql.com/doc/refman/5.6/en/upgrading-from-previous-series.html)：

![incompatible_change_of_date_and_time_type](/img/mysql/datatype/incompatible_change_of_date_and_time_type.png)

参考 [Date and Time Type Storage Requirements](https://dev.mysql.com/doc/refman/5.6/en/storage-requirements.html#data-types-storage-reqs-date-time) 下表列明了日期与时间类型在 MySQL 5.6.4 前后的变化：

![date_and_time_type_storage_requirements](/img/mysql/datatype/date_and_time_type_storage_requirements.png)

有关于时间值的内部表示的详细信息，参考 [MySQL Internals: Important Algorithms and Structures - Date and Time Data Type Representation](https://dev.mysql.com/doc/internals/en/date-and-time-data-type-representation.html)

`DATETIME` 类型非小数部分的编码如下：

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

# 参考

https://dev.mysql.com/doc/refman/5.7/en/data-types.html

https://dev.mysql.com/doc/refman/5.7/en/column-count-limit.html

《[MySQL数据类型：UNSIGNED注意事项](https://www.cnblogs.com/blankqdb/archive/2012/11/03/blank_qdb.html)》

《[MySQL 5.6时间数据类型功能获得改进](http://tech.it168.com/a2013/1013/1544/000001544067.shtml)》