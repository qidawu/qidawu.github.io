---
title: MySQL 常用日期函数
date: 2018-01-26 21:43:12
updated:
tags: MySQL
---

# 日期函数

## 获得当前日期时间

```mysql
select now();                           -- 2018-08-08 22:20:46
select CURRENT_TIMESTAMP;               -- 2018-08-08 22:20:46
select curdate();                       -- 2018-08-08
select curtime();                       -- 22:41:30
select unix_timestamp();                -- 1218290027
select unix_timestamp('2008-08-08');    -- 1218124800
```

## 日期、时间戳互转

| 函数                 | 描述                             |
| ------------------ | ------------------------------ |
| `FROM_UNIXTIME()`  | `TIMESTAMP` 类型 > `DATETIME` 类型 |
| `UNIX_TIMESTAMP()` | `TIMESTAMP` 类型 < `DATETIME` 类型 |

## 日期、字符串互转

Date/Time to Str（日期/时间转换为字符串）函数：

* `date_format(date, format)`
* `time_format(time, format)`

`format` 如下：

| `format` | 描述 |
| -------- | ---- |
| `%Y`     | 年   |
| `%m`     | 月   |
| `%d`     | 日   |
| `%H`     | 时   |
| `%i`     | 分   |
| `%s`     | 秒   |

例子：

```mysql
select date_format(now(), '%Y-%m-%d');                          -- 2018-08-08
select date_format('2018-08-08 22:23:00', '%W %M %Y');          -- Friday August 2018
select date_format('2018-08-08 22:23:01', '%Y%m%d%H%i%s');      -- 20180808222301
select time_format('22:23:01', '%H.%i.%s');                     -- 22.23.01
```

Str to Date （字符串转换为日期）函数：

* `str_to_date(str, format)`

```mysql
select str_to_date('08/09/2008', '%m/%d/%Y');                   -- 2008-08-09
select str_to_date('08/09/08'  , '%m/%d/%y');                   -- 2008-08-09
select str_to_date('08.09.2008', '%m.%d.%Y');                   -- 2008-08-09
select str_to_date('08:09:30', '%h:%i:%s');                     -- 08:09:30
select str_to_date('08.09.2008 08:09:30', '%m.%d.%Y %h:%i:%s'); -- 2008-08-09 08:09:30
```

## 日期时间计算

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

## 日期时间截取

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