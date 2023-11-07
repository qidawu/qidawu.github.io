---
title: Java 常用类型系列（五）时区处理总结
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

每个特定的 `ZoneId` 对象都有一个地区 ID 标识。地区 ID 格式为“`{区域}/{城市}`”，这些地区集合的设定都由 [IANA  的时区数据库](https://www.iana.org/time-zones)提供。可以输出如下：

```java
ZoneId.getAvailableZoneIds().forEach(System.out::println);
```

`ZoneId` 的静态工厂方法构造如下：

```java
// 获取服务器所在时区的 ZoneId，例如 Asia/Shanghai 为 UTC+8
ZoneId currentZone = ZoneId.systemDefault();

// 获取指定城市的 ZoneId，即 UTC+1
ZoneId zoneId = ZoneId.of("Europe/Paris");
```

一旦得到一个 `ZoneId` 对象，就可以与 `LocalDate`、`LocalDateTime`、`Instant` 对象整合起来，构造一个 `ZonedDateTime` 实例，它代表了**相对于指定时区的时间点**。

## ZonedDateTime

https://docs.oracle.com/javase/8/docs/api/java/time/ZonedDateTime.html

> A date-time with a time-zone in the ISO-8601 calendar system, such as `2007-12-03T10:15:30+01:00 Europe/Paris`.

Java 8 中 `ZonedDateTime` 基于 ISO-8601 实现，参考[这里](/posts/java8-time/#ISO-8601)。

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

# 常见问题

## java.sql.Timestamp

有时开发会使用 `java.sql.Timestamp` 作为 PO 实体类的时间字段，`java.sql.Timestamp` 底层实现使用**格里历（公历）**，并使用服务器所在时区（即**本地时区**），并受该时区影响。

这里看一段代码，以 2021-01-04 00:00:00 为例演示转换过程：

```java
LocalDateTime localDateTime = LocalDateTime.parse(
  "2021-01-04 00:00:00", 
  DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
);
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, ZoneId.of("Asia/Jakarta"));

Timestamp.from(zonedDateTime.toInstant());
Timestamp.valueOf(localDateTime);
```

这里试验两个时区：

|                            | Europe/London (UTC) | Asia/Shanghai (UTC+8) |
| -------------------------- | ------------------- | --------------------- |
| `ZoneId.systemDefault()`   | Europe/London (UTC) | Asia/Shanghai (UTC+8) |
| `TimeZone.getDefaultRef()` | Europe/London (UTC) | Asia/Shanghai (UTC+8) |

下面分别看下 `java.sql.Timestamp` 两个 API 会有什么问题：

### #valueOf(LocalDateTime)

转换过程：本地时间 > 系统时区的时间 > UTC-0 时区的时间戳

| Europe/London (UTC)                                          | Asia/Shanghai (UTC+8)                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 2021-01-04T00:00 (`LocalDateTime`) → <br/>2021-01-04T00:00:00.000Z / 1609718400000 (`Timestamp`) | 2021-01-04T00:00 (`LocalDateTime`) → <br/>2021-01-04T00:00:00.000+0800 / 1609689600000 (`Timestamp`) |

可见，由于 `LocalDateTime` 本身不含时区信息，在经由 `Timestamp#valueOf(LocalDateTime)` 转换时，源码中使用了 `TimeZone.getDefaultRef()` 并**受系统默认时区的影响，导致结果前后不一致**。

```java
    /**
     * Returns the reference to the default TimeZone object. This
     * method doesn't create a clone.
     */
    static TimeZone getDefaultRef() {
        TimeZone defaultZone = defaultTimeZone;
        if (defaultZone == null) {
            // Need to initialize the default time zone.
            defaultZone = setDefaultZone();
            assert defaultZone != null;
        }
        // Don't clone here.
        return defaultZone;
    }
```

### #from(Instant)

转换过程：本地时间 > 指定时区的时间 > UTC-0 时区的时间戳

| Europe/London (UTC)                                          | Asia/Shanghai (UTC+8)                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 2021-01-04T00:00 (`LocalDateTime`) → <br/>2021-01-04T00:00+07:00[Asia/Jakarta] (`ZonedDateTime`) → <br/>2021-01-03T17:00:00Z / 1609693200 (`Instant`) →<br/>2021-01-03T17:00:00.000Z / 1609693200000(`Timestamp`) | 2021-01-04T00:00 (`LocalDateTime`) → <br/>2021-01-04T00:00+07:00[Asia/Jakarta] (`ZonedDateTime`) → <br/>2021-01-03T17:00:00Z / 1609693200 (`Instant`) →<br/>2021-01-04T01:00:00.000+0800 / 1609693200000 (`Timestamp`) |

这里看似结果没有问题，`Instant` 和 `Timestamp` 对象在不同时区下都是相同时间戳。

但有一种场景，就是**应用服务器与数据库的时区不一致导致的问题**。假如应用服务器时区为 `Asia/Shanghai (UTC+8)`，数据库时区为 `Europe/London (UTC)`，当把上表 `Timestamp` 对象保存到 MySQL 数据库的 `datetime` 字段时，如果未经时区转换，会导致错误结果。

这里参考 [mysql-connector-java-5.1.42.jar](https://mvnrepository.com/artifact/mysql/mysql-connector-java) 源码如下，重点看 `java.sql.PreparedStatement#setTimestamp` 的方法实现，其使用了 `SimpleDateFormat` 将 `Timestamp` 对象格式化成字符串，**如果未经时区转换，结果如下表，导致前后不一致**：

| 格式化前                     | 格式化后            |
| ---------------------------- | ------------------- |
| 2021-01-03T17:00:00.000Z     | 2021-01-03 17:00:00 |
| 2021-01-04T01:00:00.000+0800 | 2021-01-04 01:00:00 |

```java
    /**
     * Set a parameter to a java.sql.Timestamp value. The driver converts this
     * to a SQL TIMESTAMP value when it sends it to the database.
     * 
     * @param parameterIndex
     *            the first parameter is 1, the second is 2, ...
     * @param x
     *            the parameter value
     * @param tz
     *            the timezone to use
     * 
     * @throws SQLException
     *             if a database-access error occurs.
     */
    private void setTimestampInternal(int parameterIndex, Timestamp x, Calendar targetCalendar, TimeZone tz, boolean rollForward) throws SQLException {
        ...
        
        x = TimeUtil.changeTimezone(this.connection, sessionCalendar, targetCalendar, x, tz, this.connection.getServerTimezoneTZ(), rollForward);

        ...
                    synchronized (this) {
                        if (this.tsdf == null) {
                            this.tsdf = new SimpleDateFormat("''yyyy-MM-dd HH:mm:ss", Locale.US);
                        }

                        StringBuffer buf = new StringBuffer();
                        buf.append(this.tsdf.format(x));
                        
                        ...
                        
                        setInternal(parameterIndex, buf.toString());
                    }
    }
```

上述方法内部调用了 `com.mysql.jdbc.TimeUtil#changeTimezone` 方法，源码如下。

```java
    /**
     * Change the given timestamp from one timezone to another
     * 
     * @param conn
     *            the current connection to the MySQL server
     * @param tstamp
     *            the timestamp to change
     * @param fromTz
     *            the timezone to change from
     * @param toTz
     *            the timezone to change to
     * 
     * @return the timestamp changed to the timezone 'toTz'
     */
    public static Timestamp changeTimezone(MySQLConnection conn, Calendar sessionCalendar, Calendar targetCalendar, Timestamp tstamp, TimeZone fromTz,
            TimeZone toTz, boolean rollForward) {
        if ((conn != null)) {
            // 开启 useTimezone=true，才能进入下面的时区转换逻辑
            if (conn.getUseTimezone()) {
                // Convert the timestamp from GMT to the server's timezone
                Calendar fromCal = Calendar.getInstance(fromTz);
                fromCal.setTime(tstamp);

                int fromOffset = fromCal.get(Calendar.ZONE_OFFSET) + fromCal.get(Calendar.DST_OFFSET);
                Calendar toCal = Calendar.getInstance(toTz);
                toCal.setTime(tstamp);

                int toOffset = toCal.get(Calendar.ZONE_OFFSET) + toCal.get(Calendar.DST_OFFSET);
                int offsetDiff = fromOffset - toOffset;
                long toTime = toCal.getTime().getTime();

                if (rollForward) {
                    toTime += offsetDiff;
                } else {
                    toTime -= offsetDiff;
                }

                Timestamp changedTimestamp = new Timestamp(toTime);

                return changedTimestamp;
            } else if (conn.getUseJDBCCompliantTimezoneShift()) {
                if (targetCalendar != null) {

                    Timestamp adjustedTimestamp = new Timestamp(jdbcCompliantZoneShift(sessionCalendar, targetCalendar, tstamp));

                    adjustedTimestamp.setNanos(tstamp.getNanos());

                    return adjustedTimestamp;
                }
            }
        }

        return tstamp;
    }
```

如果 JDBC 连接参数未配置 `useTimezone=true`（默认值 `false`），会导致目标时区转换失效，从而产生上述问题。**而如果开启之后，不管应用服务器设置什么时区，都能保证正确转换为数据库目标时区的时间值，反之亦然（数据库 -> 应用服务器）。**这里给两个例子，如下表：

|       | 时区转换前                                | 时区转换后                            |
| ----- | ----------------------------------------- | ------------------------------------- |
| UTC+2 | 2021-01-03T19:00:00.000+0200 / 1609693200 | 2021-01-03T17:00:00.000Z / 1609693200 |
| UTC+8 | 2021-01-04T01:00:00.000+0800 / 1609693200 | 2021-01-03T17:00:00.000Z / 1609693200 |

![after_convert_time_zone+2](/img/java/time/after_convert_time_zone+2.png)

![after_convert_time_zone+8](/img/java/time/after_convert_time_zone+8.png)

参考：

[不指定时区会踩坑：MySQL JDBC 8.0.22 驱动升级遇到的 Bug 分析](https://mp.weixin.qq.com/s/tYQ6fXH4x74pLwkbKtnDjQ)

# 参考

时区数据库：

* [IANA  的时区数据库](https://www.iana.org/time-zones)
* [《这个重要开源项目全靠一位低调的 “怪老头” 维护！他和比尔盖茨一样撑起了计算机世界》](https://www.infoq.cn/article/S7iIaYMTOecZowBxBCY3)

  > 时区设置背后有一组大量关于全球许多代表性地点时间历史信息的代码和数据，这些代码和数据被称为**时区数据库**（即 tz、tzdata 或 zoneinfo），该数据库会定期进行更新以反映各政治实体对时区边界、UTC 差值和夏令时规则的更改。对 tz 的更新遵循 BCP 175 流程进行管理。
  >
  > 尽管大多数计算机用户从未听说过时区数据库，但 tz 数据库对全世界的计算机非常重要。所有基于 Linux 和 Mac 的计算机都是从一个极其重要的数据库（时区数据库）中提取时区。目前，使用该数据库的项目包括：the GNU C Library (used in GNU/Linux), Android, FreeBSD, NetBSD, OpenBSD, Chromium OS, Cygwin, MariaDB, MINIX, MySQL, webOS, AIX, BlackBerry 10, iOS, macOS, Microsoft Windows, OpenVMS, Oracle Database, Oracle Solaris 等。
  >
  > **tz 数据库背后，一个人在维护**
  >
  > tz 数据库由 David Olson 创立，收集了自 1970 年以来被广泛认可的民用时钟的时区信息。2011 年，互联网域名与数字地址分配机构 ICANN 接管了这个被全球电脑和网站广泛使用的时区数据库，该机构通常只赞助对互联网发展非常重要的项目，
  >
  > 现在，具体的维护工作由互联网分配号码管理局（Internet Assigned Numbers Authority， IANA）负责。Paul Eggert 是时区数据库的项目负责人，该职位被称为 TZ 协调员。

* [Get Time Zone info of the World Countries](http://www.world-timedate.com/timezone/timezone_info_by_country.php)

* [逐渐成为历史的 “新疆时间” —— Asia/Urumqi (UTC+06:00)](https://zhuanlan.zhihu.com/p/352366074)

UTC：

* https://en.wikipedia.org/wiki/Coordinated_Universal_Time
* https://zh.wikipedia.org/wiki/协调世界时
* [闰秒](https://en.wikipedia.org/wiki/Leap_second)终于要被取消了！

  > 2022 年 11 月 20 日消息，[负责协调世界时的国际计量局（BIPM）](https://www.bipm.org/en/time-metrology)表示，科学家和政府代表 18 日在法国举行的一次会议上投票决定到 2035 年取消闰秒。BIPM 时间部门负责人帕特里齐亚·塔维拉表示，这项“历史性决定”将允许“秒数连续流动，而不会出现目前由不规则闰秒造成的不连续性。”
  >
  > **更多阅读：**
  >
  > [为什么闰秒调整将从 2035 年开始暂停 | InfoQ](https://mp.weixin.qq.com/s/6F6pHKrTCMKxpaMJOYK1JA)
  >
  > [正式宣布取消！能让 Linus 本人同谷歌微软达成一致的，只有它了！](https://mp.weixin.qq.com/s/yyDyRUE6_0Co_1PRjgeOCQ)
  >
  > [让大厂抓狂的“额外一秒”：谷歌、微软、Meta 和亚马逊纷纷提议放弃](https://www.infoq.cn/article/3qXArL5XGJZXmAh90ZlD)
  >
  > [闰秒这个困扰程序员 50 年的问题，终于要被解决了？](https://mp.weixin.qq.com/s/K6YGA6Dc9w3iTuiA8U-PYA)
  >
  > > 当闰秒发生时，就需要通过[网络时间协议 NTP (Network time protocol)](https://en.wikipedia.org/wiki/Network_Time_Protocol) 来进行时间同步，NTP 服务器会一级一级地下发闰秒事件通知直到最边缘的 NTP 服务器，然后 NTP 服务器就会把闰秒通知发给客户端的操作系统，由操作系统来处理闰秒通知。
  > >
  > > 如果你的计算机系统没有开启 NTP 服务，那么导致的问题就是你的计算机上的机器时间就会比世界时间慢 1 秒。
  > >
  > > 如果开了 NTP 服务的话，就需要操作系统来处理这个闰秒。

* [协调世界时（UTC） - 维基百科](https://zh.wikipedia.org/wiki/%E5%8D%8F%E8%B0%83%E4%B8%96%E7%95%8C%E6%97%B6)
* https://time.is/UTC
* [时区转换器：计算世界各个时区的时差](https://www.zeitverschiebung.net/cn/)
* [Time Zone Converter](http://www.timezoneconverter.com/cgi-bin/zoneinfo)

时间协议：

* https://en.wikipedia.org/wiki/Time_Protocol
* https://en.wikipedia.org/wiki/Network_Time_Protocol
* https://en.wikipedia.org/wiki/Network_Time_Protocol#SNTP
* 《[每天学一个 Linux 命令（96）：`ntpdate`](https://mp.weixin.qq.com/s/b5JqzO9nqmef9RdlosloPw)》


其它：

* 《Java 8 实战》
* Java SE Docs
* 《八十天环游地球》【法】儒勒·凡尔纳
* [《Java 时区数据库与 IANA 数据》](https://www.coder.work/article/6269420)
