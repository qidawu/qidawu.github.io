---
title: Java 常用类型系列（四）Java 8 日期与时间总结
date: 2018-04-05 23:39:32
updated:
tags: Java
typora-root-url: ..
---

# 现有问题

`java.util.Date` 存在的问题：

* 正如类名所表达的，这个类无法表示“日期”（替代方案是 `LocalDate`），只能以毫秒的精度表示“时间”（替代方案是 `LocalDateTime`）。
* 更糟糕的是它的易用性，比如：年份的起始选择是 1900 年，月份的起始从 0 开始。如果要表达 2014 年 3 月 18 日，需要创建以下 `Date` 实例：`Date date = new Date(144，2，18)`
* 非线程安全，且所有的日期类都是可变的，这表示在运行期其值可以任意修改，这是 Java 日期类最大的问题之一。

`java.util.Date` 及 `java.sql.Timestamp` API 如下：

![java time old api](/img/java/time/java_time_old_api.png)

# Java 8 日期与时间

Java 8 引入了新的 `java.time` 类库，用于加强对日期与时间的操作，本文主要总结其 API 结构、具体实现和使用方式。

先看下涉及的包结构简介：

| Package                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [java.time](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html) | The main API for dates, times, instants, and durations.      |
| [java.time.temporal](https://docs.oracle.com/javase/8/docs/api/java/time/temporal/package-summary.html) | Access to date and time using fields and units, and date time adjusters. |
| [java.time.chrono](https://docs.oracle.com/javase/8/docs/api/java/time/chrono/package-summary.html) | Generic API for calendar systems other than the default ISO. |
| [java.time.zone](https://docs.oracle.com/javase/8/docs/api/java/time/zone/package-summary.html) | Support for time-zones and their rules.                      |
| [java.time.format](https://docs.oracle.com/javase/8/docs/api/java/time/format/package-summary.html) | Provides classes to print and parse dates and times.         |

## java.time

`java.time` 包的核心接口：

![java-time核心接口](/img/java/time/java-time核心接口.png)

`java.time` 包的常用类：

```java
Year              // 2007
YearMonth         // 2007-12
MonthDay          // --12-03

LocalDate         // 2007-12-03
LocalTime         // 10:15:30
LocalDateTime     // 2007-12-03T10:15:30

OffsetTime        // 10:15:30+01:00
OffsetDateTime    // 2007-12-03T10:15:30+01:00
ZonedDateTime     // 2007-12-03T10:15:30+01:00 Europe/Paris

Instant           // 以 Unix 元年时间（UTC 时区 1970-01-01T00:00:00Z，“Z” 代表 UTC 时区）开始所经历的秒数
```

![java.time核心类](/img/java/time/java.time核心类.png)

## java.time.temporal

`java.time.temporal` 包的核心接口：

| Interface          | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `TemporalAccessor` | 框架级别的根接口，定义了对 temporal 对象的只读访问。         |
| `Temporal`         | 框架级别的接口，定义了对 temporal 对象的读写访问。实现类有 `LocalDate`、`OffsetDateTime`、`Instant` 等等。继承自 `TemporalAccessor`。 |
| `TemporalAmount`   | 框架级别的接口，定义了一个时间段，例如”6 小时“、”8 天“、”3 个月“。实现类有 `Duration`、`Period`。可传入 temporal 对象的 `plus` 或 `minus` 方法进行时间调整。 |
| `TemporalUnit`     | 日期和时间的单元，`ChronoUnit` 枚举实现了该接口。可传入 temporal 对象的 `plus` 或 `minus` 方法进行时间调整。 |
| `TemporalField`    | 日期和时间的字段。`ChronoField` 枚举实现了该接口，可传入 temporal 对象的 `get` 或 `with` 方法获取或修改枚举对应的值。 |
| `TemporalAdjuster` | 函数式接口，定义了对 temporal 对象的调整策略。可以使用 `TemporalAdjusters` 工具类的静态工厂方法生成对象实例，并传入temporal 对象的 `with` 方法进行时间调整。 |

![ChronoLocalDate核心接口方法](/img/java/time/temporal核心接口.png)

# LocalDate、LocalTime

`LocalDate`、`LocalTime` 是人类易读的日期和时间格式，表示一个本地时间点，**无时区信息**。

## 历法系统介绍

首先了解几个概念：

### 历法

[历法](https://zh.wikipedia.org/wiki/历法)，或称日历，是用[年](https://zh.wikipedia.org/wiki/年)、[月](https://zh.wikipedia.org/wiki/月)、[日](https://zh.wikipedia.org/wiki/日)等时间单位计算时间的方法。Java 8 提供的历法实现如下：

| 历法                                                         | Java 8 实现        |
| ------------------------------------------------------------ | ------------------ |
| [格里历（公历）](https://zh.wikipedia.org/wiki/格里曆)，即 [Gregorian calendar](https://en.wikipedia.org/wiki/Gregorian_calendar) | `LocalDate`        |
| [和历](https://zh.wikipedia.org/wiki/和历)                   | `JapaneseDate`     |
| [中华民国历](https://zh.wikipedia.org/wiki/民國紀年)         | `MinguoDate`       |
| [泰国历](https://zh.wikipedia.org/wiki/泰國曆)               | `ThaiBuddhistDate` |
| [伊斯兰历（回历）](https://zh.wikipedia.org/zh/伊斯兰历)     | `HijrahDate`       |

还有其它一些常见的历法，例如：

* [中国传统历法](https://zh.wikipedia.org/wiki/%E4%B8%AD%E5%9B%BD%E4%BC%A0%E7%BB%9F%E5%8E%86%E6%B3%95)
* [会计年度](https://zh.wikipedia.org/wiki/會計年度)：

> 财政年度，又称会计年度，是指公司或国家每年制定预算或计算收入的统计时间。但每个国家或其法例所辖的组织各有不同，大抵分成两类：
>
> * 历年制，即是由1月1日起，使用历年制有中国、德国、法国、等等。
> * 跨年制，使用跨年制有美国、日本、香港、澳大利亚、等等。
>
> 在会计上常会用 [4/4/5 日历](https://en.wikipedia.org/wiki/4-4-5_Calendar)，每一个月会有固定的周数，以便各月之间和各年之间的比较。

### 纪年

> [纪年](https://zh.wikipedia.org/wiki/纪年)，或称**纪元**，是指[历法](https://zh.wikipedia.org/wiki/历法)中的**年份**命名体系，例如[格里历](https://zh.wikipedia.org/wiki/格里曆)（公历）所使用的[基督纪年](https://zh.wikipedia.org/wiki/基督纪年)（[公元](https://zh.wikipedia.org/wiki/公元)），中国[农历](https://zh.wikipedia.org/wiki/农历)使用的[干支纪年](https://zh.wikipedia.org/wiki/干支纪年)等。世界各地曾存在过各种不同的纪年方法，其中一些至今仍在使用，例如日本现在仍在使用[年号纪年](https://zh.wikipedia.org/wiki/年号纪年)。
>
> 这里提供了一个常见的[纪年对照表](https://zh.wikipedia.org/wiki/%E6%B0%91%E5%9C%8B%E7%B4%80%E5%B9%B4#%E7%B4%80%E5%B9%B4%E5%B0%8D%E7%85%A7)，可供参考。 

> [年号](https://zh.wikipedia.org/wiki/%E5%B9%B4%E5%8F%B7)是中国历史君主时代帝王纪年所立的名号，缘起于西汉汉武帝时期，后来朝鲜新罗在6世纪、日本在7世纪后期、越南在10世纪都因为中国的影响，开始使用年号；台湾岛的郑氏王朝与台湾民主国、朝鲜半岛大韩帝国与高丽、蒙古国建国初年受到中国影响，都还使用过年号，目前唯一使用年号的是仍保持君主制的日本。
>
> 值得一提，[中华民国](https://zh.wikipedia.org/wiki/中華民國)所用的[民国纪年](https://zh.wikipedia.org/wiki/民國紀年)、以及[朝鲜民主主义人民共和国](https://zh.wikipedia.org/wiki/朝鮮民主主義人民共和國)使用的[主体纪年](https://zh.wikipedia.org/wiki/主体纪年)，常被误认为是年号，实际上仅是单纯的纪年[历法](https://zh.wikipedia.org/wiki/曆法)。

> [一世一元制](https://zh.wikipedia.org/wiki/%E4%B8%80%E4%B8%96%E4%B8%80%E5%85%83%E5%88%B6)，指[君主](https://zh.wikipedia.org/wiki/君主)(国王、大君主、可汗、天皇、皇帝)在其在位期间只使用同一个[年号](https://zh.wikipedia.org/wiki/年號)，不进行改元的制度。例外：如果君主后来另行称帝或是重祚，会另建新年号，以示区别。

### ISO-8601

[ISO-8601](https://zh.wikipedia.org/wiki/ISO_8601) 日历系统是世界文明日历系统的事实标准：

> In general, ISO 8601 applies to these representations and formats: 
>
> * *dates,* in the [Gregorian](https://en.wikipedia.org/wiki/Gregorian_calendar) calendar (including the [proleptic Gregorian](https://en.wikipedia.org/wiki/Proleptic_Gregorian_calendar) calendar); 
> * *times,* based on the [24-hour timekeeping system](https://en.wikipedia.org/wiki/24-hour_clock), with optional [UTC offset](https://en.wikipedia.org/wiki/UTC_offset); [*time intervals*](https://en.wikipedia.org/wiki/Time_interval); and combinations thereof.[[2]](https://en.wikipedia.org/wiki/ISO_8601#cite_note-scope-2) 
>
> The standard does not assign specific meaning to any element of the dates/times represented: the meaning of any element depends on the context of its use. 
>
> Dates and times represented cannot use words that do not have a specified numerical meaning within the standard (thus excluding [names of years](https://en.wikipedia.org/wiki/Chinese_calendar_correspondence_table) in the [Chinese calendar](https://en.wikipedia.org/wiki/Chinese_calendar)), or that do not use [computer characters](https://en.wikipedia.org/wiki/Character_(computing)) (excludes images or sounds).[[2]](https://en.wikipedia.org/wiki/ISO_8601#cite_note-scope-2)

ISO-8601 的**日期表示法**为：

> In representations that adhere to the ISO 8601 *interchange standard* :
>
> * dates and times are arranged such that the greatest temporal term (typically a year) is placed at the left and each successively lesser term is placed to the right of the previous term. 
> * Representations must be written in a combination of [Arabic numerals](https://en.wikipedia.org/wiki/Arabic_numerals) and the specific computer characters (such as "-", ":", "T", "W", "Z") that are assigned specific meanings within the standard; that is, such commonplace descriptors of dates (or parts of dates) as "January", "Thursday", or "New Year's Day" are not allowed in interchange representations within the standard.

> 年由 4 位数字组成 `YYYY`，或者带正负号的四或五位数字表示 `±YYYYY`：
>
> * 公元 1 年为 0001 年
> * 公元前 1 年为 0000 年
> * 公元前 2 年为 -0001 年
>
> 其它以此类推。
>
> 月、日用两位数字表示：`MM`、`DD`。

https://docs.oracle.com/javase/8/docs/api/java/time/LocalDateTime.html

> A date-time without a time-zone in the ISO-8601 calendar system, such as `2007-12-03T10:15:30`.

Java 8 中 `LocalDate` 基于 ISO-8601 实现。另外还提供了四种其它的日历系统。这些日历系统中的每一个都有一个对应的类，如下图。所有这些类都实现了 `java.time.chrono.ChronoLocalDate` 接口，能够对日期进行建模。

![ChronoLocalDate实现类](/img/java/time/ChronoLocalDate实现类.png)

```java
// 格里历（公历）
LocalDate date = LocalDate.now();
// 和历
JapaneseDate japaneseDate = JapaneseDate.from(date);
// 中华民国历
MinguoDate minguoDate = MinguoDate.from(date);
// 泰国历
ThaiBuddhistDate thaiBuddhistDate = ThaiBuddhistDate.from(date);
// 伊斯兰历（回历）
HijrahDate hijrahDate = HijrahDate.from(date);
```

## 底层实现

`LocalDate` 的底层实现包含不可变的年、月、日：

```java
public final class LocalDate
        implements Temporal, TemporalAdjuster, ChronoLocalDate, Serializable {
    /**
     * The year.
     */
    private final int year;
    /**
     * The month-of-year.
     */
    private final short month;
    /**
     * The day-of-month.
     */
    private final short day;
}
```

`LocalTime` 的底层实现包含不可变的时、分、秒、纳秒：

```java
public final class LocalTime
        implements Temporal, TemporalAdjuster, Comparable<LocalTime>, Serializable {
    /**
     * The hour.
     */
    private final byte hour;
    /**
     * The minute.
     */
    private final byte minute;
    /**
     * The second.
     */
    private final byte second;
    /**
     * The nanosecond.
     */
    private final int nano;   
}
```

`LocalDateTime` 的底层实现包含不可变的 `LocalDate` 和 `LocalTime`：

```java
public final class LocalDateTime
        implements Temporal, TemporalAdjuster, ChronoLocalDateTime<LocalDate>, Serializable {
    /**
     * The date part.
     */
    private final LocalDate date;
    /**
     * The time part.
     */
    private final LocalTime time;
}
```

## 使用方式

| 方法名     | 是否静态方法 | 描述                                                         |
| ---------- | ------------ | ------------------------------------------------------------ |
| `of`       | 是           | 由 `Temporal` 对象的某个部分创建该对象。                     |
| `now`      | 是           | 依据系统时钟创建 `Temporal` 对象。                           |
| `from`     | 是           | 依据传入的 `Temporal` 对象创建对象。                         |
| `parse`    | 是           | 由字符串创建 `Temporal` 对象。                               |
| `format`   | 否           | 使用某个指定的 `DateTimeFormatter` 将 `Temporal` 对象转换为字符串。 |
| `atOffset` | 否           | 将 `Temporal` 对象和某个时区偏移相结合。                     |
| `atZone`   | 否           | 将 `Temporal` 对象和某个时区相结合。                         |
| `get`      | 否           | 读取 `Temporal` 对象的某一部分值。                           |
| `minus`    | 否           | 创建 `Temporal` 对象的副本，通过将当前 `Temporal` 对象的值减去一定的时长创建该副本。 |
| `plus`     | 否           | 创建 `Temporal` 对象的副本，通过将当前 `Temporal` 对象的值加上一定的时长创建该副本。 |
| `with`     | 否           | 以该 `Temporal` 对象为模板，对某些状态进行修改创建该对象的副本。 |

例子：

### 创建实例

```java
// 通过静态工厂方法创建实例
LocalTime time = LocalTime.of(12, 0);
LocalDate date = LocalDate.of(2007, 12, 3);
// 1970-01-01。epoch day（历元日）是一个简单的天数递增计数，其中第 0 天表示 1970-01-01。负数代表更早的日期。
LocalDate date1 = LocalDate.ofEpochDay(0);
// 通过静态方法创建实例
LocalDate date2 = LocalDate.parse("2007-12-03");

// 通过时间戳创建实例
LocalDateTime.ofEpochSecond(timestamp, 0, ZoneOffset.of("+08:00"))

// 合并日期和时间
LocalDateTime dateTime = date.atTime(time);
LocalDateTime dateTime1 = time.atDate(date);
LocalDateTime dateTime2 = date.atStartOfDay();
LocalDateTime dateTime3 = LocalDateTime.of(date, time);

// java.time.LocalDateTime 转 java.sql.Timestamp
Timestamp timestamp = Timestamp.valueOf(localDateTime);
```

### 读取字段信息

读取信息使用 `TemporalAccessor` 接口，通过传递一个 `ChronoField` 枚举给 `get` 方法读取相关信息：

![ChronoLocalDate核心接口方法](/img/java/time/temporal核心接口.png)

`ChronoField` 枚举提供的值如下图：

![ChronoField枚举](/img/java/time/ChronoField枚举.png)：

```java
// 使用 TemporalAccessor#get(TemporalField)，传入 TemporalField 接口的实现类 ChronoField
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int dayOfMonth = date.get(ChronoField.DAY_OF_MONTH);

// LocalTime/LocalDate/LocalDateTime 各自也提供了一组 getter 快捷方法
int year1 = date.getYear();
int month1 = date.getMonthValue();
int dayOfMonth1 = date.getDayOfMonth();
boolean leapYear = date.isLeapYear();
```

### 创建副本并修改

创建副本使用 `Temporal` 接口，其提供了一组 `with`/`plus`/`minus` 方法：

![ChronoLocalDate核心接口方法](/img/java/time/temporal核心接口.png)

```java
// 使用 Temporal#with(TemporalField, long)，传入 TemporalField 接口的实现类 ChronoField
LocalDate newDate = date.with(ChronoField.YEAR, 2019);
LocalDate newDate2 = date.with(ChronoField.MONTH_OF_YEAR, 1);
LocalDate newDate3 = date.with(ChronoField.DAY_OF_MONTH, 30);

// LocalTime/LocalDate/LocalDateTime 各自也提供了一组 with 快捷方法
LocalDate newDate4 = date.withYear(2019);
LocalDate newDate5 = date.withMonth(1);
LocalDate newDate6 = date.withDayOfMonth(30);
```

如果 `with(TemporalField, long)` 方法不满足需求，可以使用更灵活的 `with(TemporalAdjuster)` 方法，配合 `TemporalAdjusters` 工具类提供的静态工厂方法，方法名非常直观：

![TemporalAdjusters静态工厂方法](/img/java/time/TemporalAdjusters静态工厂方法.png)

```java
// 使用 Temporal#with(TemporalAdjuster)
LocalDate newDate7 = date.with(TemporalAdjusters.nextOrSame(DayOfWeek.SUNDAY));
LocalDate newDate8 = date.with(TemporalAdjusters.lastDayOfMonth());
```

如果在 `TemporalAdjusters` 工具类中没有找到符合要求的预定义静态工厂方法，可以自己实现 `TemporalAdjuster` 函数式接口，以定制一些更复杂的时间修改操作：

![TemporalAdjuster](/img/java/time/TemporalAdjuster.png)

# Instant

作为人，我们习惯于以星期几、几号、几点、几分这样的方式理解日期和时间。毫无疑问，这种方式对于计算机而言并不容易理解。从计算机的角度来看，建模时间最自然的格式是表示一个持续时间段上某个点的单一大整型数。这也是新的 `java.time.Instant` 类对时间建模的方式，它是以 **Unix 元年时间**（UTC 时区 1970-01-01T00:00:00Z，“Z” 代表 UTC 时区）开始所经历的秒数进行计算。

> 1 秒(s) = 
> 1,000 毫秒(ms) = 
> 1,000,000 微秒(μs) = 
> 1,000,000,000 纳秒(ns) = 
> 1,000,000,000,000 皮秒(ps) = 
> 1,000,000,000,000,000 飞秒(fs) = 
> 1,000,000,000,000,000,000 仄秒(zs) = 
> 1,000,000,000,000,000,000,000 幺秒(ys) = 
> 1,000,000,000,000,000,000,000,000 渺秒(as)

> 1 纳秒(ns) = 10^-9 秒（0.000,000,001，十亿分之一秒）
> 1 微秒(μs) = 10^-6 秒（0.000,001，百万分之一秒）
> 1 毫秒(ms) = 10^-3 秒（0.001，千分之一秒）

## 底层实现

从底层实现可见，`Instant` 仅包含不可变的秒、纳秒。即支持的最高存储精度为**纳秒**：

```java
public final class Instant
        implements Temporal, TemporalAdjuster, Comparable<Instant>, Serializable {
    /**
     * The number of seconds from the epoch of 1970-01-01T00:00:00Z.
     */
    private final long seconds;
    /**
     * The number of nanoseconds, later along the time-line, from the seconds field.
     * This is always positive, and never exceeds 999,999,999.
     */
    private final int nanos;
}
```

## 使用方式

通过静态工厂方法创建实例：

```java
// 自 1970-01-01T00:00:00Z 之后经过的毫秒数
// 1970-01-01T00:00:01.000Z
Instant instant = Instant.ofEpochMilli(1000);

// // 自 1970-01-01T00:00:00Z 之后经过的秒数
// 1970-01-01T00:00:01Z
Instant instant1 = Instant.ofEpochSecond(1);

Instant instant2 = Instant.now();
```

`Instant` 与 `java.util.Date` 互转：

```java
Date date = Date.from(instant);
Instant instant = date.toInstant();
```

`Instant` 与时间戳互转：

```java
Instant instant = Instant.now(); // 2019-07-07T11:07:42.814Z

// Instant 的设计初衷是为了便于机器使用，底层实现仅包含由秒和纳秒所构成的数字。
long seconds = instant.getEpochSecond();  // 1562497662
int nanoSeconds = instant.getNano();  // 814000000

// Instant 无法处理那些我们非常容易理解的时间单位，例如下述操作将抛出异常：
// java.time.temporal.UnsupportedTemporalTypeException: Unsupported field: DayOfMonth
// instant.get(ChronoField.DAY_OF_MONTH);

// Instant 与时间戳互转
long timestamp = instant.toEpochMilli();  // 1562497662814
assertEquals(System.currentTimeMillis(), timestamp); // true
Instant instant2 = Instant.ofEpochMilli(timestamp); // 2019-07-07T11:07:42.814Z
```

时间字符串转 `Instant`：

```java
Instant.from(DateTimeFormatter.ISO_OFFSET_DATE_TIME.parse("2001-07-04T12:08:56+05:30"));
```

创建副本并修改：

```java
Instant newInstant = instant.with(ChronoField.INSTANT_SECONDS, 30);
Instant newInstant2 = instant.with(ChronoField.MILLI_OF_SECOND, 300);
Instant newInstant3 = instant.plus(30, ChronoUnit.SECONDS);
Instant newInstant4 = instant.plusSeconds(30);
Instant newInstant5 = instant.minusSeconds(30);
```

# Duration、Period

`Duration` 和 `Period` 类用于保存两个 `Temporal` 对象之间的时间段。

![TemporalAmount实现类](/img/java/time/TemporalAmount实现类.png)

## 底层实现

`Duration` 的底层实现仅包含不可变的秒、纳秒：

```java
public final class Duration
        implements TemporalAmount, Comparable<Duration>, Serializable {
    /**
     * The number of seconds in the duration.
     */
    private final long seconds;
    /**
     * The number of nanoseconds in the duration, expressed as a fraction of the
     * number of seconds. This is always positive, and never exceeds 999,999,999.
     */
    private final int nanos;
}
```

`Period` 的底层实现包含不可变的年、月、日：

```java
public final class Period
        implements ChronoPeriod, Serializable {
    /**
     * The number of years.
     */
    private final int years;
    /**
     * The number of months.
     */
    private final int months;
    /**
     * The number of days.
     */
    private final int days;
}
```

## 使用方式

```java
// Duration 类的静态工厂方法 between 仅支持两个 LocalTime 或两个 LocalDateTime、或两个 Instant 对象之间的 duration
Duration duration = Duration.between(dateTime1, dateTime);
Duration.between(time, time);
Duration.between(instant, instant1);
// 由于 Duration 类主要用于以秒和纳秒衡量时间的长短，如果传 LocalDate 对象会抛异常：java.time.temporal.UnsupportedTemporalTypeException
// Duration.between(date1, date);

// Period 类的静态工厂方法 between 仅支持两个 LocalDates
Period period = Period.between(date1, date);

// Duration 和 Period 类都提供了很多非常方便的静态工厂方法，用于直接创建对应的实例
Duration threeDays = Duration.ofDays(3);
Duration threeHours = Duration.ofHours(3);
Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes2 = Duration.of(3, ChronoUnit.MINUTES);
Duration threeSeconds = Duration.ofSeconds(3);
Duration threeMillis = Duration.ofMillis(3);
Duration twoDaysThreeHoursFourMinutes = Duration.parse("P2DT3H4M");

Period tenYears = Period.ofYears(10);
Period tenMonths = Period.ofMonths(10);
Period tenWeeks = Period.ofWeeks(10);
Period tenDays = Period.ofDays(10);
Period twoYearsSixMonthsOneDay = Period.of(2, 6, 1);
Period oneYearTwoMonthsThreeDays = Period.parse("P1Y2M3D");

// 准确获取该时段的差值：日、时、分、秒、毫秒、纳秒
long days = duration.toDays();
long hours = duration.toHours();
long minutes = duration.toMinutes();
long seconds = duration.getSeconds();
long seconds2 = duration.get(ChronoUnit.SECONDS);
long millis = duration.toMillis();
long nanos2 = duration.toNanos();

// 获取指定部分的差值
int years = period.getYears();
int months = period.getMonths();
int days1 = period.getDays();
long days2 = period.get(ChronoUnit.DAYS);
```

`ChronoUnit` 枚举提供的值：

![TemporalUnit枚举](/img/java/time/TemporalUnit枚举.png)

# 日期格式化

`DateTimeFormatter` 用于日期格式化。和老的 `java.util.DateFormat` 相比，所有的 `DateTimeFormatter` 实例都是线程安全的。所以，你能够以单例模式创建格式器实例，并在多个线程间共享。

下面介绍创建格式器的三种方式：

## 预定义的格式器

创建格式器最简单的方法是通过它[预定义的格式器](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#predefined)常量，定义如下：

![DateTimeFormatter常量](/img/java/time/DateTimeFormatter常量.png)

| Formatter                                                    | Description                                              | Example                                   |
| :----------------------------------------------------------- | :------------------------------------------------------- | :---------------------------------------- |
| [`ofLocalizedDate(dateStyle)`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ofLocalizedDate-java.time.format.FormatStyle-) | Formatter with date style from the locale                | '2011-12-03'                              |
| [`ofLocalizedTime(timeStyle)`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ofLocalizedTime-java.time.format.FormatStyle-) | Formatter with time style from the locale                | '10:15:30'                                |
| [`ofLocalizedDateTime(dateTimeStyle)`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ofLocalizedDateTime-java.time.format.FormatStyle-) | Formatter with a style for date and time from the locale | '3 Jun 2008 11:05:30'                     |
| [`ofLocalizedDateTime(dateStyle,timeStyle)`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ofLocalizedDateTime-java.time.format.FormatStyle-) | Formatter with date and time styles from the locale      | '3 Jun 2008 11:05'                        |
| [`BASIC_ISO_DATE`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#BASIC_ISO_DATE) | Basic ISO date                                           | '20111203'                                |
| [`ISO_LOCAL_DATE`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_LOCAL_DATE) | ISO Local Date                                           | '2011-12-03'                              |
| [`ISO_OFFSET_DATE`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_OFFSET_DATE) | ISO Date with offset                                     | '2011-12-03+01:00'                        |
| [`ISO_DATE`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_DATE) | ISO Date with or without offset                          | '2011-12-03+01:00'; '2011-12-03'          |
| [`ISO_LOCAL_TIME`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_LOCAL_TIME) | Time without offset                                      | '10:15:30'                                |
| [`ISO_OFFSET_TIME`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_OFFSET_TIME) | Time with offset                                         | '10:15:30+01:00'                          |
| [`ISO_TIME`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_TIME) | Time with or without offset                              | '10:15:30+01:00'; '10:15:30'              |
| [`ISO_LOCAL_DATE_TIME`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_LOCAL_DATE_TIME) | ISO Local Date and Time                                  | '2011-12-03T10:15:30'                     |
| [`ISO_OFFSET_DATE_TIME`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_OFFSET_DATE_TIME) | Date Time with Offset                                    | 2011-12-03T10:15:30+01:00'                |
| [`ISO_ZONED_DATE_TIME`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_ZONED_DATE_TIME) | Zoned Date Time                                          | '2011-12-03T10:15:30+01:00[Europe/Paris]' |
| [`ISO_DATE_TIME`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_DATE_TIME) | Date and time with ZoneId                                | '2011-12-03T10:15:30+01:00[Europe/Paris]' |
| [`ISO_ORDINAL_DATE`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_ORDINAL_DATE) | Year and day of year                                     | '2012-337'                                |
| [`ISO_WEEK_DATE`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_WEEK_DATE) | Year and Week                                            | 2012-W48-6'                               |
| [`ISO_INSTANT`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_INSTANT) | Date and Time of an Instant                              | '2011-12-03T10:15:30Z'                    |
| [`RFC_1123_DATE_TIME`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#RFC_1123_DATE_TIME) | RFC 1123 / RFC 822                                       | 'Tue, 3 Jun 2008 11:05:30 GMT'            |

例子：

```java
// 2021-02-01
LocalDateTime.now().format(DateTimeFormatter.ISO_DATE);
// 2021-02-01T09:55:54.846Z（注意：UTC+00:00 输出为 Z，需要自行 replace）
ZonedDateTime.now(ZoneId.of("Europe/London")).format(DateTimeFormatter.ISO_OFFSET_DATE_TIME);
// 2021-02-01T16:43:46+07:00
ZonedDateTime.now(ZoneId.of("Asia/Jakarta")).truncatedTo(ChronoUnit.SECONDS).format(DateTimeFormatter.ISO_OFFSET_DATE_TIME);
```

## 自定义 Pattern

[Patterns for Formatting and Parsing](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#patterns):

> Patterns are based on a simple sequence of letters and symbols. A pattern is used to create a Formatter using the [`ofPattern(String)`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ofPattern-java.lang.String-) and [`ofPattern(String, Locale)`](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ofPattern-java.lang.String-java.util.Locale-) methods.
>
> A formatter created from a pattern can be used as many times as necessary, it is immutable and is thread-safe.

`DateTimeFormatter#ofPattern(String)` 静态工厂方法：

```java
// 01/02/2021
LocalDateTime.now().format(DateTimeFormatter.ofPattern("dd/MM/yyyy"));
```

`DateTimeFormatter#ofPattern(String, Locale)` 静态工厂方法：

```java
// 2021 2 1
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy M d", Locale.US));
// 2021 02 1
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy MM d", Locale.US));
// 2021 Feb 1
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy MMM d", Locale.US));
// 2021 February 1
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy MMMM d", Locale.US));
// 2021 F 1
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy MMMMM d", Locale.US));
```

Pattern 的字母数量决定格式，以月份为例：

* Exactly 1 pattern letter will use the minimum number of digits and without padding.
* Exactly 2 pattern letters, the count of digits is used as the width of the output field, with the value zero-padded as necessary.
* Exactly 3 pattern letters will use the [`short form`](https://docs.oracle.com/javase/8/docs/api/java/time/format/TextStyle.html#SHORT).
* Exactly 4 pattern letters will use the [`full form`](https://docs.oracle.com/javase/8/docs/api/java/time/format/TextStyle.html#FULL). 
* Exactly 5 pattern letters will use the [`narrow form`](https://docs.oracle.com/javase/8/docs/api/java/time/format/TextStyle.html#NARROW).

## 更灵活的构建器

如果还需要更加细粒度的控制，`DateTimeFormatterBuilder` 类还提供了更复杂的格式器构建，你可以选择恰当的方法，一步一步地构造自己的格式器：

```java
DateTimeFormatter dtf = new DateTimeFormatterBuilder().appendPattern("dd/MM/yyyy[ [HH][:mm][:ss][.SSS]]")
  .parseDefaulting(ChronoField.HOUR_OF_DAY, 0)
  .parseDefaulting(ChronoField.MINUTE_OF_HOUR, 0)
  .parseDefaulting(ChronoField.SECOND_OF_MINUTE, 0)
  .toFormatter(Locale.US);

// 1999-01-01T02:04:06
LocalDateTime.parse("01/01/1999 02:04:06", dtf);
// 1999-01-01T00:00
LocalDateTime.parse("01/01/1999", dtf);
```

另外，`DateTimeFormatterBuilder` 类还提供了非常强大的解析功能，比如区分大小写的解析、柔性解析（允许解析器使用启发式的机制去解析输入，不精确地匹配指定的模式）、填充，以及在格式器中指定可选节。

# 参考

* 《Java 8 实战》
* Java SE Docs
* [ISO 8601 - DATE AND TIME FORMAT](https://www.iso.org/iso-8601-date-and-time-format.html)
* [RFC 3339 - Date and Time on the Internet: Timestamps](https://datatracker.ietf.org/doc/html/rfc3339)
* [What's the difference between ISO 8601 and RFC 3339 Date Formats?](https://stackoverflow.com/questions/522251/whats-the-difference-between-iso-8601-and-rfc-3339-date-formats)
* 《[Parsing and Formatting (The Java™ Tutorials > Date Time > Standard Calendar)](https://docs.oracle.com/javase/tutorial/datetime/iso/format.html)》
* 《[`uuuu` versus `yyyy` in `DateTimeFormatter` formatting pattern codes in Java?](https://stackoverflow.com/questions/41177442/uuuu-versus-yyyy-in-datetimeformatter-formatting-pattern-codes-in-java)》
* 《[LocalDate、LocalDateTime与timestamp、Date的转换](https://www.jianshu.com/p/b4629857fc6f)》
