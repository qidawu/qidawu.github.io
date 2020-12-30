---
title: MySQL 常用日期函数
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

## 时区

### 查看当前时区
```sql
-- 结果主要看 system_time_zone
show variables like '%time_zone%';

-- 查询系统时区和会话时区
SELECT @@GLOBAL.time_zone, @@SESSION.time_zone;
```

参考：[MySQL 中几个关于时间/时区的变量](https://www.cnblogs.com/Uest/p/8259821.html)

### 修改时区

通过 SQL `SET` 语法临时修改：

```sql
set global time_zone = '+8:00';  -- 设置 Global 全局时区，重启后失效
set time_zone = '+8:00';         -- 设置 Session 会话时区，会话关闭后失效
```

通过修改配置文件，重启后永久生效：

```sql
$ vim /etc/mysql/my.cnf
default-time_zone = '+8:00'

$ service mysql restart
```

### 时区转换

参考：https://dev.mysql.com/doc/refman/5.7/en/datetime.html

> MySQL converts `TIMESTAMP` values from the current time zone to UTC for storage, and back from UTC to the current time zone for retrieval. (This does not occur for other types such as `DATETIME`.) By default, the current time zone for each connection is the server's time. The time zone can be set on a per-connection basis. As long as the time zone setting remains constant, you get back the same value you store. If you store a `TIMESTAMP` value, and then change the time zone and retrieve the value, the retrieved value is different from the value you stored. This occurs because the same time zone was not used for conversion in both directions. The current time zone is available as the value of the [`time_zone`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_time_zone) system variable. For more information, see [Section 5.1.13, “MySQL Server Time Zone Support”](https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html).

`CONVERT_TZ(dt, from_tz, to_tz)` 函数用于将 `DATETIME` 类型转为指定时区，例如：

```SQL
# TIMESTAMP 类型（UTC±00:00） > DATETIME 类型（UTC±00:00） > DATETIME 类型（UTC+08:00）
SELECT CONVERT_TZ( FROM_UNIXTIME( UNIX_TIMESTAMP() ), '+00:00', '+08:00' ) AS NOW;
```

## 日期/时间转换

### 日期/时间 > 时间戳

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`UNIX_TIMESTAMP([date])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_unix-timestamp) | Return a Unix timestamp.<br/>If no *`date`* argument, it returns a Unix timestamp representing seconds since `'1970-01-01 00:00:00'` UTC.<br/>If with a *`date`* argument, it returns the value of the argument as seconds since `'1970-01-01 00:00:00'` UTC. |

例子：

```sql
SELECT UNIX_TIMESTAMP();                -- 1218124800，获取当前时间戳
SELECT UNIX_TIMESTAMP(now());           -- 1218124800，将当前时间转换为时间戳，等价于上例
SELECT UNIX_TIMESTAMP('2008-08-08');    -- 1219125100，将指定参数转换为时间戳
```

### 时间戳 > 日期/时间

| Name                                                         | Description                     |
| ------------------------------------------------------------ | ------------------------------- |
| [`FROM_UNIXTIME(unix_timestamp[,format])`](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_from-unixtime) | Format Unix timestamp as a date |

注意，该函数只支持单位为秒的时间戳，不支持毫秒、微秒，需要先换算：

```sql
SELECT FROM_UNIXTIME(1447430881);                                    -- 2015-11-13 10:08:01
SELECT FROM_UNIXTIME(1447430881123 / 1000);                          -- 2015-11-13 16:08:01.1230
SELECT FROM_UNIXTIME(1447430881123456 / 1000000, '%Y %D %M %r %f');  -- 2015 13th November 04:08:01 PM 123456
```

支持指定 `fomart` 格式：

```sql
SELECT FROM_UNIXTIME(0);                       -- 1970-01-01 00:00:00
SELECT FROM_UNIXTIME(0, '%Y %D %M %h:%i:%s');  -- 1970 1st January 12:00:00
```

### 日期/时间 > 字符串

Date/Time to Str（日期/时间转换为字符串）函数：

* `date_format(date, format)`
* `time_format(time, format)`

例子：

```mysql
select date_format(now(), '%Y-%m-%d');                          -- 2018-08-08
select date_format('2018-08-08 22:23:00', '%W %M %Y');          -- Friday August 2018
select date_format('2018-08-08 22:23:01', '%Y%m%d%H%i%s');      -- 20180808222301
select time_format('22:23:01', '%H.%i.%s');                     -- 22.23.01
```

### 字符串 > 日期/时间

Str to Date （字符串转换为日期）函数：

* `str_to_date(str, format)`

```mysql
select str_to_date('08/09/2008', '%m/%d/%Y');                   -- 2008-08-09
select str_to_date('08/09/08'  , '%m/%d/%y');                   -- 2008-08-09
select str_to_date('08.09.2008', '%m.%d.%Y');                   -- 2008-08-09
select str_to_date('08:09:30', '%h:%i:%s');                     -- 08:09:30
select str_to_date('08.09.2008 08:09:30', '%m.%d.%Y %h:%i:%s'); -- 2008-08-09 08:09:30
```

### format 参数

`format` 如下，这里只列出常用的：

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

## 日期/时间计算

为日期增加一个时间间隔：

* `date_add()`

为日期减去一个时间间隔：

* `date_sub()`

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

## 日期/时间截取

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
select count(*), DATE_FORMAT(create_time,'%Y-%m-%d') hours 
from t_order 
where create_time BETWEEN '2008-9-29' AND '2008-9-30' 
group by hours;

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

做法在于将每行的分钟数 `(MINUTE(create_time)` 除以 10 得到的小数向下取整，再乘以 10 就是所属的区间。例如：

* 1 分钟 > 0
* 25 分钟 > 20
* 59 分钟 > 50

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

# 建表自动初始化与更新时间

假设表有 3 个字段：id、name、update_time，希望在新增记录时能自动设置 update_time 字段为当前时间，设置 `DEFAULT CURRENT_TIMESTAMP` 子句即可：

```mysql
CREATE TABLE `test` (
`id` int NOT NULL,
`name` varchar(255),
`update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
PRIMARY KEY (`id`)
);
```

如果希望在更新记录时还能自动更新 update_time 字段为当前时间，设置 `ON UPDATE CURRENT_TIMESTAMP` 子句即可：

```mysql
...
`update_time` timestamp NOT NULL ON UPDATE CURRENT_TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
...
```

设置之后，即使直接通过 Navicat 工具修改了 name 字段，那么 update_time 也会自动更新，除非手动设置了 update_time 字段。

参考：https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html

# DATETIME 类型快速对比

```mysql
select count(1) 
from t_table 
where createTime between 20180215 and 20180216;
```

datetime 会自动转为整数，查询范围从 2018-02-15 00:00:00 到 2018-02-16 00:00:00。

# 参考

https://dev.mysql.com/doc/refman/5.7/en/functions.html

https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html

https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html

https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html