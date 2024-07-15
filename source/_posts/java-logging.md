---
title: Java 日志框架总结
date: 2021-01-03 22:07:11
updated: 
tags: Java
typora-root-url: ..
---

日志门面：

- Apache Commons Logging (JCL) 是 Apache Commons 的一个开源项目。官网：https://commons.apache.org/proper/commons-logging/
- SLF4J 全称为 Simple Logging Facade for Java，即 Java 简单日志门面。官网：http://www.slf4j.org/

# 日志门面——SLF4J

优点：

- 通过依赖配置，在编译期静态绑定真正的日志框架库。

  > [**What has changed in SLF4J version 2.0.0?**](http://www.slf4j.org/faq.html#changesInVersion200)
  >
  > More visibly, slf4j-api now relies on the [ServiceLoader](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html) mechanism to find its logging backend. SLF4J 1.7.x and earlier versions relied on the static binder mechanism which is no loger honored by slf4j-api version 2.0.x. More specifically, when initializing the `LoggerFactory` class will no longer search for the `StaticLoggerBinder` class on the class path.
  >
  > Instead of "bindings" now `org.slf4j.LoggerFactory` searches for "providers". These ship for example with *slf4j-nop-2.0.x.jar*, *slf4j-simple-2.0.x.jar* or *slf4j-jdk14-2.0.x.jar*.

- 不需要使用 `logger.isDebugEnabled()` 来解决日志因为字符拼接产生的性能问题。SLF4J 的方式是使用 `{}` 作为字符串替换符。

- 提供了很多[桥接方案](http://www.slf4j.org/manual.html#swapping)，以便灵活替换日志库。


缺点：

- 旧版不支持 Lambda 表达式（slf4j-api-2.0.0-alpha 支持但还未 GA）。但日志框架 Log4j 2.4 及以上版本支持 Java 8 Lambda，参考 [1](https://logging.apache.org/log4j/2.x/manual/api.html#LambdaSupport)、[2](https://www.baeldung.com/log4j-2-lazy-logging)，例如：

  ```java
  logger.debug("This {} and {} with {} ", () -> this, () -> that, () -> compute());
  ```

## SLF4J bindings

http://www.slf4j.org/manual.html#swapping

![concrete bindings](/img/java/logging/concrete-bindings.png)

### Logback

https://logback.qos.ch/

推荐使用 SLF4J bound to logback-classic 的经典组合：

```XML
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.2.3</version>
</dependency>
```

传递依赖如下：

```
ch.qos.logback:logback-classic:jar:1.2.3:compile
+- ch.qos.logback:logback-core:jar:1.2.3:compile
\- org.slf4j:slf4j-api:jar:1.7.25:compile
```

### Log4j 2.x

https://logging.apache.org/log4j/2.x/

需引入适配层依赖：

```XML
<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.16.0</version>
</dependency>
```

传递依赖如下：

```
org.apache.logging.log4j:log4j-slf4j-impl:jar:2.16.0:compile
+- org.slf4j:slf4j-api:jar:1.7.25:compile
+- org.apache.logging.log4j:log4j-api:jar:2.16.0:compile
\- org.apache.logging.log4j:log4j-core:jar:2.16.0:runtime
```

参考：

* 《[试试这款性能最强的日志框架 Log4j 2](https://mp.weixin.qq.com/s/HCbTSvacZCwwleCdbrcGMw)》
* <[Reasons to prefer logback over log4j 1.x](https://logback.qos.ch/reasonsToSwitch.html)>
  * [*log4j.properties* to *logback.xml* Translator](https://logback.qos.ch/translator/)
* 《[突发！Log4j 2 爆“核弹级”漏洞，Flink、Kafka 等至少十多个项目受影响](https://mp.weixin.qq.com/s/Yq9k1eBquz3mM1sCinneiA)》
* 《[Log4j 2 再发新版本 2.16.0，完全删除 Message Lookups 的支持，加固漏洞防御！](https://mp.weixin.qq.com/s/lTlZ_vAy0_Vo15UpFl4jSw)》

### Log4j 1.x

https://logging.apache.org/log4j/1.2/

> On August 5, 2015 the Logging Services Project Management Committee announced that Log4j 1.x had reached end of life.

需引入适配层依赖：

```XML
<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.32</version>
</dependency>
```

传递依赖如下：

```
org.slf4j:slf4j-log4j12:jar:1.7.32:compile
+- org.slf4j:slf4j-api:jar:1.7.32:compile
\- log4j:log4j:jar:1.2.17:compile
```

### java.util.logging (JUL)

需引入适配层依赖：

```XML
<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-jdk14 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jdk14</artifactId>
    <version>1.7.32</version>
</dependency>
```

传递依赖如下：

```
org.slf4j:slf4j-jdk14:jar:1.7.32:compile
\- org.slf4j:slf4j-api:jar:1.7.32:compile
```

# 日志框架——Logback

## Configuration

https://logback.qos.ch/manual/configuration.html

![Configuration file syntax](/img/java/logging/logback/basicSyntax.png)

![Configuring Appenders](/img/java/logging/logback/appenderSyntax.png)

Logback 配置文件样例：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <springProperty scope="context" name="LOG_PATH" source="log.path" defaultValue="/data/logs"/>
    <springProperty scope="context" name="ACTIVE_PROFILE" source="spring.profiles.active" defaultValue="dev" />
    <springProperty scope="context" name="LOG_PATTERN" source="log.pattern" defaultValue="[%d{yyyy-MM-dd HH:mm:ss.SSS}] %5p [%t] %logger{50}:%L [%X{traceId}] --> %m%n" />
    <springProperty scope="context" name="MONGO_URI" source="spring.data.mongodb.uri" />

    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <!-- 每天归档日志文件 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/project-${ACTIVE_PROFILE}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <fileNamePattern>${LOG_PATH}/project-${ACTIVE_PROFILE}-%i.log.%d{yyyyMMdd}</fileNamePattern>
            <!--日志文件保留天数-->
            <maxHistory>90</maxHistory>
            <maxFileSize>100MB</maxFileSize>
            <totalSizeCap>20GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <!-- 自定义 MongoDB Appender -->
    <appender name="MONGO" class="com.test.MongoDBAppender"></appender>

    <logger name="org.springframework" level="WARN"/>

    <root level="INFO">
        <appender-ref ref="FILE" />
        <appender-ref ref="STDOUT" />
        <appender-ref ref="MONGO" />
    </root>

</configuration>
```

## Appenders

有三种 Appenders：

1. Logback 官方提供的 Appenders。
2. 第三方提供的 Appenders（logstash、…）。
3. 自定义 Appenders。

### 官方 Appenders

Logback 官方提供的 Appenders 如下：http://logback.qos.ch/manual/appenders.html

![Appender](/img/java/logging/logback/Appender.png)

其中，常用的两个 Appender 继承结构如下：

* `ch.qos.logback.core.ConsoleAppender`
* `ch.qos.logback.core.rolling.RollingFileAppender`

![OutputStreamAppender](/img/java/logging/logback/OutputStreamAppender.png)

其中，`ch.qos.logback.core.rolling.RollingFileAppender` 可选的策略类如下：

![Policy](/img/java/logging/logback/Policy.png)

### 第三方 Appenders

https://github.com/logstash/logstash-logback-encoder

> Provides [logback](http://logback.qos.ch/) encoders, layouts, and appenders to log in JSON and [other formats supported by Jackson](https://github.com/logstash/logstash-logback-encoder#data-format).

### 自定义 Appenders

通过自定义 Appenders，可以将日志输出到任意位置，如 MongoDB、Kafka 等服务。

下面演示一个简单的同步 Appender，如需异步 Appender，参考[这里](http://logback.qos.ch/manual/appenders.html#AsyncAppender)。

首先，新建 Appender，继承自抽象类：

![UnsynchronizedAppenderBase](/img/java/logging/logback/UnsynchronizedAppenderBase.png)

![ILoggingEvent](/img/java/logging/logback/ILoggingEvent.png)

代码如下：

```java
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.UnsynchronizedAppenderBase;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoClientDbFactory;

public class MongoDBAppender extends UnsynchronizedAppenderBase<ILoggingEvent> {

    private SimpleMongoClientDbFactory mongoDbFactory;
    private MongoTemplate mongoTemplate;
    private final ReentrantLock lock = new ReentrantLock(false);

    @Override
    public void start() {
        String uri = this.getContext().getProperty("MONGO_URI");
        mongoDbFactory = new SimpleMongoClientDbFactory(uri);
        mongoTemplate = new MongoTemplate(mongoDbFactory);
        super.start();
    }

    @Override
    protected void append(ILoggingEvent eventObject) {
        lock.lock();
        try {
            PayLog log = new PayLog();
            log.setTraceId(eventObject.getMDCPropertyMap().getOrDefault(MDCUtils.LOG_TRACE_ID, null));
            log.setOper(eventObject.getMDCPropertyMap().getOrDefault(MDCUtils.OPER, null));
            log.setLevel(eventObject.getLevel().toString());
            log.setLogger(eventObject.getLoggerName());
            log.setThread(eventObject.getThreadName());
            log.setMessage(eventObject.getFormattedMessage());
            log.setTimestamp(eventObject.getTimeStamp());
            mongoTemplate.insert(log, "payLog");
        } finally {
            lock.unlock();
        }
    }

}
```

最后，验证 MongoDB 数据。

## Encoders & Layouts

https://logback.qos.ch/manual/encoders.html

https://logback.qos.ch/manual/layouts.html

Logback 提供的 `ch.qos.logback.core.encoder.Encoder` 和 `ch.qos.logback.core.Layout` 的默认实现如下：

![Layout](/img/java/logging/logback/Encoder_and_Layout.png)

一般有几种使用组合方式：

### 自定义 Pattern 格式

基于自定义 [Pattern](http://logback.qos.ch/manual/layouts.html#PatternLayout) 格式，直接使用 `ch.qos.logback.classic.encoder.PatternLayoutEncoder` 即可：

```XML
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
  ...
  <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
    <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
  </encoder>
</appender>
```

还可以输出扩展信息，例如 `%X{traceId}`。

### 预定义格式

基于预定义格式，需要使用 `ch.qos.logback.core.encoder.LayoutWrappingEncoder`，并搭配相应 `Layout` 实现。例如输出成 HTML 格式：

```XML
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    ...
    <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
        <layout class="ch.qos.logback.classic.html.HTMLLayout" />
    </encoder>
</appender>
```

## Mapped Diagnostic Contexts (MDC)

https://logback.qos.ch/manual/mdc.html

> To uniquely stamp each request, the user puts contextual information into the `MDC`, the abbreviation of *Mapped Diagnostic Context*.
>
> The `MDC` class contains only static methods. It lets the developer place information in a *diagnostic context* that can be subsequently retrieved by certain logback components. The `MDC` manages contextual information on a *per thread basis*. Typically, while starting to service a new client request, the developer will insert pertinent contextual information, such as the client id, client's IP address, request parameters etc. into the `MDC`. Logback components, if appropriately configured, will automatically include this information in each log entry.
>
> Please note that MDC as implemented by logback-classic assumes that values are placed into the MDC with moderate frequency. Also note that a child thread does not automatically inherit a copy of the mapped diagnostic context of its parent.
>
> Logback leverages SLF4J API:
>
> * [org.slf4j.MDC](https://www.slf4j.org/api/org/slf4j/MDC.html)
> * [org.slf4j.MDC.MDCCloseable](https://www.slf4j.org/api/org/slf4j/MDC.MDCCloseable.html)

### MDC And Managed Threads

https://logback.qos.ch/manual/mdc.html#managedThreads

### `MDCInsertingServletFilter`

https://logback.qos.ch/manual/mdc.html#mis

### 例子

一、创建 MDCUtils

```java
import lombok.experimental.UtilityClass;
import org.slf4j.MDC;

@UtilityClass
public class MDCUtils {

    public static final String TRACE_ID = "traceId";

    public MDC.MDCCloseable addTraceId(String traceId) {
        return MDC.putCloseable(TRACE_ID, traceId);
    }

}
```

二、在合适的位置设置上下文：

- 在 http 接口的拦截器
- 在定时任务执行之前
- ...

代码如下：

```java
try (MDC.MDCCloseable mdc = MDCUtils.addTraceId(...)) {
  // TODO
}
```

![TraceIdInterceptor](/img/java/logging/logback/TraceIdInterceptor.png)

三、选择合适的 Appender 并加以配置，参考 Logback 配置文件样例。

# 参考

https://github.com/logstash/logstash-logback-encoder

《[细说 Java 主流日志工具库](https://mp.weixin.qq.com/s/68QncZycXgWPWgYg71wZGQ)》

《[SpringBoot + MDC 实现全链路调用日志跟踪](https://juejin.cn/post/6844904101483020295)》

《[Java 中打印日志的正确姿势](https://mp.weixin.qq.com/s/v3aF1zVwsP5g8KcuJFP4fQ)》

《[实战总结 | 系统日志规范及最佳实践 | 阿里技术](https://mp.weixin.qq.com/s/V-TIT1Cw5fH8xSYAEMyukQ)》

《[浅析 JAVA 日志中的几则性能实践与原理解释 | 阿里技术](https://mp.weixin.qq.com/s/uhlBOIHrDuE2P8GX1gam6w)》