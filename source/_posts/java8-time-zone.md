---
title: Java 常用类型系列（五）Java 8 时区处理总结
date: 2018-04-06 19:39:32
updated:
tags: Java
typora-root-url: ..
---

本文介绍时区处理的两种方式。时区涉及的接口及实现类如下：

![classes_of_time_zone](/img/java/time/classes_of_time_zone.png)

（上图简化掉了 `Serializable` 接口、`Comparable` 接口及 `FunctionalInterface` 注解）

# 推荐方式

## ZoneId

时区的处理是新版日期与时间 API 新增的重要功能，且 API 被极大简化。新的 `java.time.ZoneId` 类是老版本 `java.util.TimeZone` 类的替代品。它的设计目标就是要让用户无需为时区处理的复杂和繁琐而操心，比如处理夏令时（DST）问题。

每个特定的 `ZoneId` 对象都有一个地区 ID 标识。地区 ID 格式为“`{区域}/{城市}`”，这些地区集合的设定都由 [IANA  的时区数据库](https://www.iana.org/time-zones)提供。静态工厂方法构造如下：

```java
// 获取服务器所在时区的 ZoneId，例如 Asia/Shanghai 为 UTC+8
ZoneId currentZone = ZoneId.systemDefault();

// 获取指定城市的 ZoneId，即 UTC+1
ZoneId zoneId = ZoneId.of("Europe/Paris");
```

一旦得到一个 `ZoneId` 对象，就可以与 `LocalDate`、`LocalDateTime`、`Instant` 对象整合起来，构造一个 `ZonedDateTime` 实例，它代表了**相对于指定时区的时间点**。

## ZonedDateTime

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

`ZonedDateTime` 的实例如下图：

![instance_of_ZonedDateTime](/img/java/time/instance_of_ZonedDateTime.png)

图中可见，原本 `LocalDateTime` 对象作为一个**本地**日期与时间，是不包含时区信息的，即没有时区概念。而在结合了 `ZoneId` 构造成一个 `ZonedDateTime` 实例之后，才有了时区概念。它代表了**相对于指定时区的时间点**。

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

# 不推荐方式

## ZoneOffset

另一种比较通用的表达时区的方式是利用当前时区和 UTC/格林尼治的固定偏差。可以使用 `ZoneOffset` 类，它是 `ZoneId` 的一个子类，表示的是当前时间和 UTC 的偏差：

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
```

## OffsetDateTime

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

# 参考

* 《Java 8 实战》
* Java SE Docs
* [协调世界时（UTC） - 维基百科](https://zh.wikipedia.org/wiki/%E5%8D%8F%E8%B0%83%E4%B8%96%E7%95%8C%E6%97%B6)
* [时区转换器：计算世界各个时区的时差](https://www.zeitverschiebung.net/cn/)
* https://time.is/UTC
* [Time Zone Converter](http://www.timezoneconverter.com/cgi-bin/zoneinfo)
* [《Java时区数据库与IANA数据》](https://www.coder.work/article/6269420)
