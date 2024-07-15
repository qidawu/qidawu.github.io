---
title: MySQL 常用日期与时间函数
date: 2018-01-26 21:43:12
updated:
tags: MySQL
---

# 日期函数

## 获得当前日期/时间

| Name                                                         | Synonym                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`CURRENT_DATE`, `CURRENT_DATE()`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_current-date) | [`CURDATE()`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_curdate) | Return the current date as <br/>*`'YYYY-MM-DD'`* / *`YYYYMMDD`* |
| [`CURRENT_TIME`, `CURRENT_TIME([fsp])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_current-time) | [`CURTIME([fsp])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_curtime) | Return the current time as <br/>*`'hh:mm:ss'`* / *`hhmmss`*<br/>The value is expressed in the session time zone. |
| [`CURRENT_TIMESTAMP`, `CURRENT_TIMESTAMP([fsp])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_current-timestamp)<br/>[`LOCALTIME`, `LOCALTIME([fsp])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_localtime)<br/>[`LOCALTIMESTAMP`, `LOCALTIMESTAMP([fsp])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_localtimestamp) | [`NOW([fsp])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_now) | Return the current date and time as <br/>*`'YYYY-MM-DD hh:mm:ss'`* / *`YYYYMMDDhhmmss`*<br/>The value is expressed in the session time zone. |
| [`UTC_DATE`, `UTC_DATE()`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_utc-date) |                                                              | Return the current UTC date as <br/>*`'YYYY-MM-DD'`* / *`YYYYMMDD`* |
| [`UTC_TIME`, `UTC_TIME([fsp])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_utc-time) |                                                              | Return the current UTC time as <br/>*`'hh:mm:ss'`* / *`hhmmss`* |
| [`UTC_TIMESTAMP`, `UTC_TIMESTAMP([fsp])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_utc-timestamp) |                                                              | Return the current UTC date and time as <br/>*`'YYYY-MM-DD hh:mm:ss'`* / *`YYYYMMDDhhmmss`* |

注意，MySQL 时间支持的最高存储精度为微秒：

> 1 秒(s) = 
> 1,000 毫秒(ms) = 
> 1,000,000 微秒(μs) = 
> 1,000,000,000 纳秒(ns) = 
> 1,000,000,000,000 皮秒(ps) = 
> 1,000,000,000,000,000 飞秒(fs) = 
> 1,000,000,000,000,000,000 仄秒(zs) = 
> 1,000,000,000,000,000,000,000 幺秒(ys) = 
> 1,000,000,000,000,000,000,000,000 渺秒(as)

> 1 微秒(μs) = 10^-6 秒（0.000,001，百万分之一秒）
> 1 毫秒(ms) = 10^-3 秒（0.001，千分之一秒）

因此 `fsp` 参数范围只能为 0 ~ 6：

> If the *`fsp`* argument is given to specify a fractional seconds precision from 0 to 6, the return value includes a fractional seconds part of that many digits.

例子：

```mysql
SELECT CURDATE();                       -- 2018-08-08，获取当前年月日
SELECT CURTIME();                       -- 22:41:30，获取当前时分秒
SELECT NOW();                           -- 2018-08-08 22:20:46，获取当前年月日时分秒
SELECT NOW(3);                          -- 2018-08-08 22:20:46.166，获取当前年月日时分秒毫秒
SELECT NOW(6);                          -- 2018-08-08 22:20:46.166123，获取当前年月日时分秒毫秒微秒
SELECT CURRENT_TIMESTAMP;               -- 2018-08-08 22:20:46，获取当前年月日时分秒
```

## 时区查看/修改

[5.1.13 MySQL Server Time Zone Support](https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html)

### 查看当前时区
```sql
-- 结果主要看 system_time_zone
show variables like '%time_zone%';

-- 查询系统时区、会话时区、下一事务时区
SELECT @@GLOBAL.time_zone, 
       @@SESSION.time_zone, 
       @@time_zone;;
```

参考：[MySQL 中几个关于时间/时区的变量](https://www.cnblogs.com/Uest/p/8259821.html)

### 修改时区

通过 SQL `SET` 语法临时修改：

```sql
-- 设置 Global 全局时区，重启后失效
set global time_zone = '+8:00';

-- 设置 Session 会话时区，会话关闭后失效
set time_zone = '+8:00';

-- 设置 下一事务 时区，事务结束后失效
set @@time_zone = '+8:00';
```

通过修改配置文件，重启后永久生效：

```sql
$ vim /etc/mysql/my.cnf
default-time_zone = '+8:00'

$ service mysql restart
```

## 时区转换 函数

`CONVERT_TZ(dt, from_tz, to_tz)` 函数用于将 `DATETIME` 类型转为指定时区，例如：

```SQL
-- TIMESTAMP 类型
-- 1218124800，获取当前时间戳
SELECT UNIX_TIMESTAMP();

-- DATETIME 类型
-- 2008-08-07 16:00:00 UTC±00:00
SELECT FROM_UNIXTIME( UNIX_TIMESTAMP() );

-- TIMESTAMP 类型 > DATETIME 类型 > DATETIME 类型
-- 2008-08-08 00:00:00 UTC+08:00
SELECT CONVERT_TZ( FROM_UNIXTIME( UNIX_TIMESTAMP() ), '+00:00', '+08:00' ) AS NOW;
```

```SQL
-- DATETIME 类型
-- 2008-08-08 00:00:00 UTC+08:00
SELECT CONVERT_TZ( '2008-08-07 16:00:00', '+00:00', '+08:00' );
```

`TIMESTAMP` 类型的时区显示，参考：https://dev.mysql.com/doc/refman/5.7/en/datetime.html

> MySQL converts `TIMESTAMP` values from the current time zone to UTC for storage, and back from UTC to the current time zone for retrieval. (This does not occur for other types such as `DATETIME`.) By default, the current time zone for each connection is the server's time. The time zone can be set on a per-connection basis. As long as the time zone setting remains constant, you get back the same value you store. If you store a `TIMESTAMP` value, and then change the time zone and retrieve the value, the retrieved value is different from the value you stored. This occurs because the same time zone was not used for conversion in both directions. The current time zone is available as the value of the [`time_zone`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_time_zone) system variable. For more information, see [Section 5.1.13, “MySQL Server Time Zone Support”](https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html).

## 日期/时间类型转换 函数

### TIMESTAMP → xxx

[`FROM_UNIXTIME(unix_timestamp[,format])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_from-unixtime)

> Returns a representation of *`unix_timestamp`* as a [`DATETIME`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) or [`VARCHAR`](https://dev.mysql.com/doc/refman/5.7/en/char.html) value. The value returned is expressed using the session time zone. (Clients can set the session time zone as described in [Section 5.1.13, “MySQL Server Time Zone Support”](https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html).) *`unix_timestamp`* is an internal timestamp value representing seconds since `'1970-01-01 00:00:00'` UTC, such as produced by the [`UNIX_TIMESTAMP()`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_unix-timestamp) function.
>
> *`unix_timestamp`*
>
> * When *`unix_timestamp`* is an integer, the fractional seconds precision of the `DATETIME` is `0`.
> * When *`unix_timestamp`* is a decimal value, the fractional seconds precision of the `DATETIME` is the same as the precision of the decimal value, up to a maximum of `6`.
> * When *`unix_timestamp`* is a floating point number, the fractional seconds precision of the datetime is `6`.
>
> *`format`*
>
> * If *`format`* is omitted, the value returned is a [`DATETIME`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html).
> * If *`format`* is supplied, the value returned is a [`VARCHAR`](https://dev.mysql.com/doc/refman/5.7/en/char.html).

例子：

```sql
-- 不指定 format 格式，返回 DATETIME 类型
SELECT FROM_UNIXTIME(1447430881);
 -> '2015-11-13 10:08:01'

-- 只支持单位为秒的时间戳，不支持毫秒、微秒，需要先除以其精度转为浮点数
SELECT FROM_UNIXTIME(1447430881123 / 1000);
 -> '2015-11-13 16:08:01.1230'

-- 运算后，转为整数类型
SELECT FROM_UNIXTIME(1447430881) + 0;
 -> 20151113100801

-- 指定 format 格式，返回 VARCHAR 类型
SELECT FROM_UNIXTIME(1447430881, '%Y %D %M %h:%i:%s %x');
 -> '2015 13th November 10:08:01 2015'
```

`fomart` 参数参考[这里](/posts/mysql-date-and-time-functions/#format-参数)。

### xxx → TIMESTAMP

[`UNIX_TIMESTAMP([date])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_unix-timestamp)

> Return a Unix timestamp.
>
> If no *`date`* argument, it returns a Unix timestamp representing seconds since `'1970-01-01 00:00:00'` UTC.
>
> If with a *`date`* argument, it returns the value of the argument as seconds since `'1970-01-01 00:00:00'` UTC.
>
> The server interprets *`date`* as a value in the session time zone and converts it to an internal Unix timestamp value in UTC. (Clients can set the session time zone as described in [Section 5.1.13, “MySQL Server Time Zone Support”](https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html).)
>
> The *`date`* argument may be 
>
> * a [`DATE`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html), [`DATETIME`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html), or [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) **string**, 
> * or a **number** in *`YYMMDD`*, *`YYMMDDhhmmss`*, *`YYYYMMDD`*, or *`YYYYMMDDhhmmss`* format. 
>
> If the argument includes a time part, it may optionally include a fractional seconds part.
>
> The return value is 
>
> * an **integer** if no argument is given or the argument does not include a fractional seconds part, 
> * or [`DECIMAL`](https://dev.mysql.com/doc/refman/5.7/en/fixed-point-types.html) if an argument is given that includes a fractional seconds part.
>
> When the *`date`* argument is a [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) column, [`UNIX_TIMESTAMP()`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_unix-timestamp) returns the internal timestamp value directly, with no implicit “string-to-Unix-timestamp” conversion.
>
> The valid range of argument values is the same as for the [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) data type: `'1970-01-01 00:00:01.000000'` UTC to `'2038-01-19 03:14:07.999999'` UTC. If you pass an out-of-range date to [`UNIX_TIMESTAMP()`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_unix-timestamp), it returns `0`.

例子：

```sql
-- 1218124800，获取当前时间戳
SELECT UNIX_TIMESTAMP();
-- 1218124800，将当前时间转换为时间戳，等价于上例
SELECT UNIX_TIMESTAMP(now());
-- 1218153600，即：2008-08-08 00:00:00 UTC
SELECT UNIX_TIMESTAMP('2008-08-08 00:00:00');
-- 1218128400，即：2008-08-07 17:00:00 UTC
SELECT UNIX_TIMESTAMP( CONVERT_TZ( '2008-08-08 00:00:00', '+07:00', '+00:00' ) );
```

### DATETIME → String

Date/Time to Str（日期/时间转换为字符串）函数：

[`DATE_FORMAT(date, format)`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_date-format)、[`TIME_FORMAT(time, format)`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_time-format)

例子：

```mysql
select date_format(now(), '%Y-%m-%d');                          -- 2018-08-08
select date_format('2018-08-08 22:23:00', '%W %M %Y');          -- Friday August 2018
select date_format('2018-08-08 22:23:01', '%Y%m%d%H%i%s');      -- 20180808222301
select time_format('22:23:01', '%H.%i.%s');                     -- 22.23.01
```

### String → DATETIME

[`STR_TO_DATE(str, format)`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_str-to-date)

> This is the inverse of the [`DATE_FORMAT()`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_date-format) function. It takes a string *`str`* and a format string *`format`*. [`STR_TO_DATE()`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_str-to-date) returns a [`DATETIME`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) value if the format string contains both date and time parts, or a [`DATE`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html) or [`TIME`](https://dev.mysql.com/doc/refman/5.7/en/time.html) value if the string contains only date or time parts. If the date, time, or datetime value extracted from *`str`* is illegal, [`STR_TO_DATE()`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_str-to-date) returns `NULL` and produces a warning.

例子：

```mysql
select str_to_date('08/09/2008', '%m/%d/%Y');                   -- 2008-08-09
select str_to_date('08/09/08'  , '%m/%d/%y');                   -- 2008-08-09
select str_to_date('08.09.2008', '%m.%d.%Y');                   -- 2008-08-09
select str_to_date('08:09:30', '%h:%i:%s');                     -- 08:09:30
select str_to_date('08.09.2008 08:09:30', '%m.%d.%Y %h:%i:%s'); -- 2008-08-09 08:09:30
```

### format 参数

`format` 参数如下，这里只列出常用的。更多 `format` 参数参考：

https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_date-format

| `format` | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| `%Y`     | Year, numeric, four digits                                   |
| `%y`     | Year, numeric (two digits)                                   |
| `%M`     | Month name (`January`..`December`)                           |
| `%m`     | Month, numeric (`00`..`12`)                                  |
| `%D`     | Day of the month with English suffix (`0th`, `1st`, `2nd`, `3rd`, …) |
| `%d`     | Day of the month, numeric (`00`..`31`)                       |
| `%H`     | Hour (`00`..`23`)                                            |
| `%h`     | Hour (`01`..`12`)                                            |
| `%i`     | Minutes, numeric (`00`..`59`)                                |
| `%s`     | Seconds (`00`..`59`)                                         |
| `%f`     | Microseconds (`000000`..`999999`)                            |
| `%T`     | Time, 24-hour (*`hh:mm:ss`*)                                 |
| `%r`     | Time, 12-hour (*`hh:mm:ss`* followed by `AM` or `PM`)        |

## 日期/时间计算 函数

为日期增加一个时间间隔：`date_add()`

为日期减去一个时间间隔：`date_sub()`

```mysql
set @dt = now();

select date_add(@dt, interval 1 day);        -- add 1 day
select date_add(@dt, interval 1 hour);       -- add 1 hour
select date_add(@dt, interval 1 minute);     -- ...
select date_add(@dt, interval 1 second);
select date_add(@dt, interval 1 microsecond);
select date_add(@dt, interval 1 week);
select date_add(@dt, interval 1 month);
select date_add(@dt, interval 1 quarter);
select date_add(@dt, interval 1 year);

select date_add(@dt, interval -1 day);       -- sub 1 day

SELECT DATE_SUB(@dt, INTERVAL 7 DAY);        -- 七天前
```

## 日期/时间截取 函数

选取日期时间的各个部分：日期、时间、年、季度、月、日、小时、分钟、秒、微秒

```mysql
set @dt = now();

select date(@dt);        -- 2008-09-10
select time(@dt);        -- 07:15:30.123456
select year(@dt);        -- 2008
select quarter(@dt);     -- 3
select month(@dt);       -- 9
select week(@dt);        -- 36
select day(@dt);         -- 10
select hour(@dt);        -- 7
select minute(@dt);      -- 15
select second(@dt);      -- 30
select microsecond(@dt); -- 123456
```

# 例子

## 按天统计订单量

按天统计订单量（同理，按小时统计：`DATE_FORMAT(create_time,'%Y-%m-%d %H:00:00')`）：

```sql
select count(*), DATE_FORMAT(create_time,'%Y-%m-%d') days 
from t_order 
where create_time BETWEEN '2008-9-29' AND '2008-9-30' 
group by days;

+----------+------------+
| count(*) | hours      |
+----------+------------+
|   150628 | 2008-09-01 |
|   172419 | 2008-09-02 |
|   177021 | 2008-09-03 |
|   178917 | 2008-09-04 |
|   180960 | 2008-09-05 |
|   177626 | 2008-09-06 |
|   177504 | 2008-09-07 |
|   166118 | 2008-09-08 |
|   193006 | 2008-09-09 |
|   204156 | 2008-09-10 |
|   196598 | 2008-09-11 |
|   200184 | 2008-09-12 |
|   159169 | 2008-09-13 |
|   179798 | 2008-09-14 |
|   203586 | 2008-09-15 |
|   217863 | 2008-09-16 |
|   231207 | 2008-09-17 |
|   245960 | 2008-09-18 |
|   226578 | 2008-09-19 |
|   211986 | 2008-09-20 |
|   201396 | 2008-09-21 |
|   183012 | 2008-09-22 |
|   221780 | 2008-09-23 |
|   228094 | 2008-09-24 |
|   220251 | 2008-09-25 |
|   240866 | 2008-09-26 |
|   235670 | 2008-09-27 |
|   244964 | 2008-09-28 |
|    98805 | 2008-09-29 |
+----------+------------+
29 rows in set (5.83 sec)
```

## 按 N 分钟统计订单量

做法在于将每行的分钟数 `MINUTE(create_time)` 除以 10 得到的小数使用 `FLOOR` 函数向下取整，再乘以 10 得到的就是所属区间。例如：

* 1 分钟 -> 0
* 25 分钟 -> 20
* 59 分钟 -> 50

下例按 10 分钟统计订单量：

```sql
SELECT COUNT(*), DATE_FORMAT(
  CONCAT(
    DATE(create_time), 
    ' ', 
    HOUR(create_time), 
    ':', 
    FLOOR(MINUTE(create_time) / 10) * 10
  ), '%Y-%m-%d %H:%i'
) AS hours
FROM t_order
WHERE create_time BETWEEN '2008-9-29' AND '2008-9-30'
GROUP BY hours;

+----------+------------------+
| COUNT(*) | hours            |
+----------+------------------+
|       51 | 2008-09-29 00:00 |
|       53 | 2008-09-29 00:10 |
|       43 | 2008-09-29 00:20 |
|       59 | 2008-09-29 00:30 |
|       40 | 2008-09-29 00:40 |
|       27 | 2008-09-29 00:50 |
|       22 | 2008-09-29 01:00 |
|       24 | 2008-09-29 01:10 |
|       15 | 2008-09-29 01:20 |
|       15 | 2008-09-29 01:30 |
|       26 | 2008-09-29 01:40 |
|       18 | 2008-09-29 01:50 |
|     1949 | 2008-09-29 02:00 |
```

# 参考

https://dev.mysql.com/doc/refman/5.7/en/functions.html

https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html

https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html

https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html