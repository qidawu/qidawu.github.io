---
title: UNIX 时间戳总结
date: 2020-02-28 22:23:38
updated:
tags: GNU/Linux
typora-root-url: ..
---

# UNIX 时间戳

![unix_timestamp](/img/gnu-linux/unix_timestamp.png)

# 时间戳问题

## Y2K (Year 2000 problem)

![Y2K](/img/gnu-linux/Y2K.jpeg)

> Y2K 是一个合成词汇：Y = Year，2 = 2，K= Kilo，因此 Y2K 的含义其实就是千禧之年 ——2000 年

https://en.wikipedia.org/wiki/Year_2000_problem

## Y2K38 (Year 2038 problem)

2038 年问题又叫 Unix 千年虫或 Y2K38 问题。在时间值以带符号的 32 位整数来存储或计算的数据存储情况下，这个错误就有可能引发问题。

下面这个动画显示了 Y2K38 问题将如何重置日期：

![Y2K38](/img/gnu-linux/Y2K38.GIF)

这是因为：用 Unix 带符号的 32 位整数时间格式来表示的最大时间是 2038 年 1 月 19 日 03:14:07UTC（2038-01-19T03:14:07Z），这是自 1970-01-01T00:00:00Z 之后过了 2147483647 秒，值的边界如下：

| 时间                 | 时间戳             | 二进制字面量                        |
| -------------------- | ------------------ | ----------------------------------- |
| 1970-01-01T00:00:00Z | 0                  | 00000000 00000000 00000000 00000000 |
| 2038-01-19T03:14:07Z | 2^31-1, 2147483647 | 01111111 11111111 11111111 11111111 |

测试代码：

```Java
// 0
long a = 0;
// 2^31-1, 2147483647
long b = Integer.MAX_VALUE;

// 1970-01-01T00:00:00.000Z
Instant.ofEpochSecond(a).atZone(ZoneOffset.of("-00:00")).toLocalDateTime()
// 2038-01-19T03:14:07.000Z
Instant.ofEpochSecond(b).atZone(ZoneOffset.of("-00:00")).toLocalDateTime()
```

过了最大时间后，由于整数溢出，时间值将作为负数来存储，系统会将日期读为 1901 年 12 月 13 日，而不是 2038 年 1 月 19 日。

用简单的语言来说，Unix 机器最终将会耗尽存储空间来列举秒数。所以，到那一天，使用标准时间库的 C程序会开始出现日期问题。你可以在[维基百科](https://en.wikipedia.org/wiki/Year_2038_problem)上详细阅读更多的相关内容：

> 目前，2038年错误没有什么通行的解决方案。如果对用于存储时间值的time_t数据类型的定义进行更改，依赖带符号的32位time_t整数性质的应用程序就会出现一些代码兼容性问题。假设time_t的类型被更改为不带符号的32位整数，那将加大最新的时间限制。但是，这会对由负整数表示的1970年之前的日期造成混乱。
>
> 使用64位架构的操作系统和程序使用64位time_t整数。使用带符号的64位值可以将日期延长至今后的2920亿年。
>
> 已有人提出了许多建议，包括以带符号的64位整数来存储自某个时间点（1970年1月1日或2000年1月1日）以来的毫秒/微秒，以获得至少30万年的时间范围。其他建议包括用新的库重新编译程序，等等。这方面的工作正在开展之中；据专家们声称，2038年问题解决起来应该不难。

# 各种开发语言获取当前时间戳

Java

[`java.time.Instant`](/posts/java8-time/#Instant)

```java
// 秒时间戳
Instant.now().getEpochSecond()
// 毫秒时间戳
Instant.now().toEpochMilli()

// 毫秒时间戳
System.currentTimeMillis()
```

Javascript

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date

```javascript
Math.round(new Date() / 1000)
```

Shell

```shell
date +%s
```

MySQL

[`UNIX_TIMESTAMP([date])`](/posts/mysql-date-and-time-functions/#DATETIME-→-TIMESTAMP)

```sql
SELECT UNIX_TIMESTAMP()
```

其它语言：...

# 参考

[UNIX时间 - 维基百科](https://zh.wikipedia.org/zh/UNIX%E6%97%B6%E9%97%B4)

Y2K problem

- https://en.wikipedia.org/wiki/Year_2000_problem
- [Y2K 千年虫（计算机 2000 年问题）](https://baike.baidu.com/item/%E5%8D%83%E5%B9%B4%E8%99%AB/2954)
- [Y2K 千年虫——一个 Bug 让人类科技倒退几十年？|良许 Linux](https://mp.weixin.qq.com/s/PayHgx8ifLsnJPuqxJr_GA)

Y2K38 problem

- https://en.wikipedia.org/wiki/Year_2038_problem
- [Unix Epoch Time（Unix 纪元时间）的 Y2K38 问题](https://mp.weixin.qq.com/s/RoSRwTLsZApvQjU7Fpy0vw)

Y2K22 problem

- [微软也栽了，“千年虫”啥时候是个头 | InfoQ](https://mp.weixin.qq.com/s/6YHSZavAv4GC-cpDja4YjQ)
- [微软开年就出大 bug——Y2K22 bug，大量程序员连夜加班：年都没跨好](https://mp.weixin.qq.com/s/QvWnzZ5-thbDFXyYBs1iQA)

[计算机时间到底是怎么来的？程序员必看的时间知识！](https://mp.weixin.qq.com/s/Xw-CQV0QvxhKw0zMgbHpQA) 