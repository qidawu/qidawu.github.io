---
title: Java 常用类型系列（四）Java 8 日期与时间总结
date: 2018-04-05 23:39:32
updated:
tags: Java
typora-root-url: ..
---

`java.util.Date` 存在的问题：

* 正如类名所表达的，这个类无法表示“日期”（替代方案是 `LocalDate`），只能以毫秒的精度表示“时间”（替代方案是 `LocalDateTime`）。
* 更糟糕的是它的易用性，比如：年份的起始选择是 1900 年，月份的起始从 0 开始。如果要表达 2014 年 3 月 18 日，需要创建以下 `Date` 实例：`Date date = new Date(144，2，18)`
* 非线程安全，且所有的日期类都是可变的，这表示在运行期其值可以任意修改，这是 Java 日期类最大的问题之一。

Java 8 引入了新的 `java.time` 类库，用于加强对日期与时间的操作，本文主要总结其 API 结构、具体实现和使用方式。

# API 结构

先看下涉及的包结构简介：

| Package                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [java.time](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html) | The main API for dates, times, instants, and durations.      |
| [java.time.temporal](https://docs.oracle.com/javase/8/docs/api/java/time/temporal/package-summary.html) | Access to date and time using fields and units, and date time adjusters. |
| [java.time.chrono](https://docs.oracle.com/javase/8/docs/api/java/time/chrono/package-summary.html) | Generic API for calendar systems other than the default ISO. |
| [java.time.zone](https://docs.oracle.com/javase/8/docs/api/java/time/zone/package-summary.html) | Support for time-zones and their rules.                      |
| [java.time.format](https://docs.oracle.com/javase/8/docs/api/java/time/format/package-summary.html) | Provides classes to print and parse dates and times.         |

`java.time` 的核心接口：

![java-time核心接口](/img/java/time/java-time核心接口.png)

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

# LocalDate、LocalTime

`LocalDate`、`LocalTime` 是人类易读的日期和时间格式，表示一个本地时间点。

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

`DateTimeFormatter` 用于日期格式化。和老的 `java.util.DateFormat` 相比，所有的 `DateTimeFormatter` 实例都是线程安全的。所以，你能够以单例模式创建格式器实例，并在多个线程间共享。创建格式器最简单的方法是通过它的常量，定义如下：

![DateTimeFormatter常量](/img/java/time/DateTimeFormatter常量.png)

`DateTimeFormatter` 还支持通过静态工厂方法创建，如下：

```java
// 2007-12-03
String format = date.format(DateTimeFormatter.ISO_DATE);
// 03/12/2007
String format2 = date.format(DateTimeFormatter.ofPattern("dd/MM/yyyy"));
// 2007 十二月 3
String format3 = date.format(DateTimeFormatter.ofPattern("yyyy MMMM d", Locale.CHINA));

// 20071114160942
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"));
```

如果还需要更加细粒度的控制，`DateTimeFormatterBuilder` 类还提供了更复杂的格式器构建，你可以选择恰当的方法，一步一步地构造自己的格式器。另外，它还提供了非常强大的解析功能，比如区分大小写的解析、柔性解析（允许解析器使用启发式的机制去解析输入，不精确地匹配指定的模式）、填充，以及在格式器中指定可选节。

# 处理不同的时区

## ZoneId、ZonedDateTime

上述日期与时间都不包含时区信息。时区的处理是新版日期与时间 API 新增的重要功能，且 API 被极大简化。新的 `java.time.ZoneId` 类是老版本 `java.util.TimeZone` 类的替代品。它的设计目标就是要让用户无需为时区处理的复杂和繁琐而操心，比如处理夏令时（DST）问题。

每个特定的 `ZoneId` 对象都由一个地区 ID 标识。地区 ID 格式为“`{区域}/{城市}`”，这些地区集合的设定都由 [IANA  的时区数据库](https://www.iana.org/time-zones)提供。静态工厂方法构造如下：

```java
// 获取服务器所在时区的 ZoneId，例如 Asia/Shanghai 为 UTC+8
ZoneId currentZone = ZoneId.systemDefault();
// 获取指定城市的 ZoneId，即 UTC+1
ZoneId zoneId = ZoneId.of("Europe/Paris");
```

一旦得到一个 `ZoneId` 对象，就可以与 `LocalDate`、`LocalDateTime`、`Instant` 对象整合起来，构造一个 `ZonedDateTime` 实例，它代表了**相对于指定时区的时间点**。

### 底层实现

`ZonedDateTime` 的底层实现如下：

```java
public final class ZonedDateTime
        implements Temporal, ChronoZonedDateTime<LocalDate>, Serializable {
    /**
     * The local date-time.
     */
    private final LocalDateTime dateTime;
    /**
     * The offset from UTC/Greenwich.
     */
    private final ZoneOffset offset;
    /**
     * The time-zone.
     */
    private final ZoneId zone;
}
```

### 使用方式

```java
// 1970-01-01
LocalDate date = LocalDate.ofEpochDay(0);
// 1970-01-01T00:00
LocalDateTime dateTime = date.atStartOfDay();
// 1970-01-01T00:00:00Z
Instant instant = Instant.ofEpochSecond(0);

// 1970-01-01T00:00+01:00[Europe/Paris]，底层调用 ZonedDateTime.of(this, zoneId)
ZonedDateTime zonedDateTime = date.atStartOfDay(zoneId);
// 1970-01-01T00:00+01:00[Europe/Paris]，底层调用 ZonedDateTime.of(this, zoneId)
ZonedDateTime zonedDateTime1 = dateTime.atZone(zoneId);
// 1970-01-01T01:00+01:00[Europe/Paris]，底层调用 ZonedDateTime.ofInstant(this, zoneId)
ZonedDateTime zonedDateTime2 = instant.atZone(zoneId);
// 2015-12-03T10:15:30+08:00[Asia/Shanghai]
ZonedDateTime zonedDateTime3 = ZonedDateTime.parse("2015-12-03T10:15:30+05:30[Asia/Shanghai]");
```

### LocalDateTime 与 Instant 互转

通过 `ZoneId` 可以将 `LocalDateTime` 和 `Instant` 进行互转，公式为 UTC + 时区差（东正西负）= 本地时间。

`LocalDateTime` > `Instant`：

```java
// 东八区的 1970-01-01T00:00，等于 UTC+0 的 1969-12-31T16:00:00Z
Instant instant2 = dateTime.atZone(ZoneId.systemDefault()).toInstant();
```

`Instant` > `LocalDateTime`：

```java
// 1970-01-01T00:00
LocalDateTime dateTime2 = LocalDateTime.ofInstant(instant2, ZoneId.systemDefault());
// 1970-01-01T08:00
LocalDateTime dateTime3 = instant.atZone(ZoneId.systemDefault()).toLocalDateTime();
```

## ZoneOffset、OffsetDateTime

另一种比较通用的表达时区的方式是利用当前时区和 UTC/格林尼治的固定偏差。可以使用 `ZoneOffset` 类，它是 `ZoneId` 的一个子类，表示的是当前时间和 UTC 的偏差：

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
```

### 底层实现

`ZoneOffset` 类可用于构造 `OffsetDateTime` 实例。`OffsetDateTime` 的底层实现如下：

```java
public final class OffsetDateTime
        implements Temporal, TemporalAdjuster, Comparable<OffsetDateTime>, Serializable {
    /**
     * The local date-time.
     */
    private final LocalDateTime dateTime;
    /**
     * The offset from UTC/Greenwich.
     */
    private final ZoneOffset offset;
}
```

### 使用方式

```java
// 1970-01-01T00:00-05:00
OffsetDateTime dateTimeInNewYork = OffsetDateTime.of(dateTime1, newYorkOffset); 
// 1970-01-01T00:00-05:00
OffsetDateTime dateTimeInNewYork2 = dateTime1.atOffset(newYorkOffset);
```

“-05:00” 的偏差实际上对应的是美国东部标准时间。注意，使用这种方式定义的 `ZoneOffset` 并未考虑任何夏令时的影响，所以在大多数情况下，不推荐使用。

# 其它历法系统

[历法](https://zh.wikipedia.org/wiki/历法)，又称日历，是用[年](https://zh.wikipedia.org/wiki/年)、[月](https://zh.wikipedia.org/wiki/月)、[日](https://zh.wikipedia.org/wiki/日)等时间单位计算时间的方法。

[ISO-8601](https://zh.wikipedia.org/wiki/ISO_8601) 日历系统（实现类 `LocalDate`）是世界文明日历系统的事实标准。但是，Java 8 中另外还提供了四种其它的日历系统。这些日历系统中的每一个都有一个对应的类，如下图。所有这些类都实现了 `java.time.chrono.ChronoLocalDate` 接口，能够对公历的日期进行建模。

![ChronoLocalDate实现类](/img/java/time/ChronoLocalDate实现类.png)

```java
LocalDate date = LocalDate.now();
JapaneseDate japaneseDate = JapaneseDate.from(date);
MinguoDate minguoDate = MinguoDate.from(date);
ThaiBuddhistDate thaiBuddhistDate = ThaiBuddhistDate.from(date);
HijrahDate hijrahDate = HijrahDate.from(date);
```

# 参考

* 《Java 8 实战》
* Java SE Docs
* [协调世界时（UTC） - 维基百科](https://zh.wikipedia.org/wiki/%E5%8D%8F%E8%B0%83%E4%B8%96%E7%95%8C%E6%97%B6)
* [时区转换器：计算世界各个时区的时差](https://www.zeitverschiebung.net/cn/)
* https://time.is/UTC
* 《[LocalDate、LocalDateTime与timestamp、Date的转换](https://www.jianshu.com/p/b4629857fc6f)》

常见历法：

* [格里历（公历）](https://zh.wikipedia.org/wiki/格里曆)
* [和历](https://zh.wikipedia.org/wiki/和历)
* [中华民国历](https://zh.wikipedia.org/wiki/民國紀年)
* [泰国历](https://zh.wikipedia.org/wiki/泰國曆)
* [伊斯兰历（回历）](https://zh.wikipedia.org/zh/伊斯兰历)

[纪年](https://zh.wikipedia.org/wiki/纪年)，或称**纪元**，是指[历法](https://zh.wikipedia.org/wiki/历法)中的年份命名体系，例如[格里历](https://zh.wikipedia.org/wiki/格里曆)（公历）所使用的[基督纪年](https://zh.wikipedia.org/wiki/基督纪年)（[公元](https://zh.wikipedia.org/wiki/公元)），中国[农历](https://zh.wikipedia.org/wiki/农历)使用的[干支纪年](https://zh.wikipedia.org/wiki/干支纪年)等。世界各地曾存在过各种不同的纪年方法，其中一些至今仍在使用。