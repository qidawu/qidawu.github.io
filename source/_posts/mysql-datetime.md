---
title: MySQL 常用日期操作
date: 2018-02-26 21:43:12
updated:
tags: MySQL
---

# 获得当前日期时间

```mysql
select now();                           -- 2018-08-08 22:20:46
select CURRENT_TIMESTAMP;               -- 2018-08-08 22:20:46
select curdate();                       -- 2018-08-08
select curtime();                       -- 22:41:30
select unix_timestamp();                -- 1218290027
select unix_timestamp('2008-08-08');    -- 1218124800
```

# 自动初始化与更新时间

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

详见：《[TIMESTAMP和DATETIME的自动初始化和更新](https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html)》

# DATETIME 类型快速对比

```mysql
select count(1) 
from t_table 
where createTime between 20180215 and 20180216;
```

datetime 会自动转为整数，查询范围从 2018-02-15 00:00:00 到 2018-02-16 00:00:00。

# 日期与时间戳互转

| 函数                 | 描述                             |
| ------------------ | ------------------------------ |
| `FROM_UNIXTIME()`  | `TIMESTAMP` 类型 > `DATETIME` 类型 |
| `UNIX_TIMESTAMP()` | `TIMESTAMP` 类型 < `DATETIME` 类型 |

# 字符串转换为日期

Str to Date （字符串转换为日期）函数：`str_to_date(str, format)`

```mysql
select str_to_date('08/09/2008', '%m/%d/%Y');                   -- 2008-08-09
select str_to_date('08/09/08'  , '%m/%d/%y');                   -- 2008-08-09
select str_to_date('08.09.2008', '%m.%d.%Y');                   -- 2008-08-09
select str_to_date('08:09:30', '%h:%i:%s');                     -- 08:09:30
select str_to_date('08.09.2008 08:09:30', '%m.%d.%Y %h:%i:%s'); -- 2008-08-09 08:09:30
```

# 日期/时间转换为字符串

Date/Time to Str（日期/时间转换为字符串）函数：`date_format(date, format)`, `time_format(time, format)`

```mysql
select date_format('2018-08-08 22:23:00', '%W %M %Y');          -- Friday August 2018
select date_format('2018-08-08 22:23:01', '%Y%m%d%H%i%s');      -- 20180808222301
select time_format('22:23:01', '%H.%i.%s');                     -- 22.23.01
```

# 日期时间截取

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

# 日期时间计算

为日期增加一个时间间隔：`date_add()`

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
```

参考

https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html