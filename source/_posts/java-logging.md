---
title: Java 日志框架总结
date: 2021-01-03 22:07:11
updated: 
tags: Java
typora-root-url: ..
---

日志门面：

- Apache Commons Logging 是 Apache Commons 的一个开源项目。官网：https://commons.apache.org/proper/commons-logging/
- SLF4J 全称为 Simple Logging Facade for Java，即 Java 简单日志门面。官网：http://www.slf4j.org/

日志实现

- java.util.logging (JUL)
- logback。http://logback.qos.ch/
- ~~Log4j 1.x。https://logging.apache.org/log4j/1.2/~~ (On August 5, 2015 the Logging Services Project Management Committee announced that Log4j 1.x had reached end of life)
- Log4j 2.x。https://logging.apache.org/log4j/2.x/

# 日志门面——SLF4J

优点：

- 通过依赖配置，在编译期静态绑定真正的日志实现库。

- 不需要使用 `logger.isDebugEnabled()` 来解决日志因为字符拼接产生的性能问题。SLF4J 的方式是使用 `{}` 作为字符串替换符。

- 提供了很多桥接方案，以便灵活替换日志库。如下：

  ![SLF4J](/img/java/logging/SLF4J.png)

缺点：

- 旧版不支持 Lambda 表达式（slf4j-api-2.0.0-alpha 支持但还未 GA）。Log4j 2.4 及以上版本支持 Java 8 Lambda，参考 [1](https://logging.apache.org/log4j/2.x/manual/api.html#LambdaSupport)、[2](https://www.baeldung.com/log4j-2-lazy-logging)，例如：

  ```java
  logger.debug("This {} and {} with {} ", () -> this, () -> that, () -> compute());
  ```

# 日志实现——Logback

## 引入依赖

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
logback-classic: 1.2.3 [compile]
  logback-core: 1.2.3 [compile]
  slf4j-api: 1.7.25 [compile]
```

## 配置 Appenders

有三种使用 Appenders 的方式：

1. 使用第一方提供的 Appenders（`ConsoleAppender`、`RollingFileAppender`）

2. 使用第三方提供的 Appenders（logstash）。

3. 使用自定义 Appenders。

### 第一方 Appenders

http://logback.qos.ch/manual/appenders.html

官方提供的第一方 Appender 如下：

![Appender](/img/java/logging/logback/Appender.png)

其中，常用的两个 Appender 如下：

![OutputStreamAppender](/img/java/logging/logback/OutputStreamAppender.png)

其中，`RollingFileAppender` 可选的策略类如下：

![Policy](/img/java/logging/logback/Policy.png)

### 第三方 Appenders

https://github.com/logstash/logstash-logback-encoder

> Provides [logback](http://logback.qos.ch/) encoders, layouts, and appenders to log in JSON and [other formats supported by Jackson](https://github.com/logstash/logstash-logback-encoder#data-format).

### 自定义 Appenders

通过自定义 Appenders，可以将日志输出到如 MongoDB、Kafka 等服务。

下面演示一个简单的同步 Appender，如需异步 Appender，参考[这里](http://logback.qos.ch/manual/appenders.html#AsyncAppender)。

一、新建 Appender，继承自抽象类：

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

二、创建 MDCUtils

![MDC](/img/java/logging/logback/MDC.png)

```java
import lombok.experimental.UtilityClass;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.MDC;

import java.util.UUID;

@UtilityClass
public class MDCUtils {

    public static final String TRACE_ID = "traceId";

    /**
     * 将 traceId 放到 MDC 中。如果 traceId 入参为空，则直接用 UUID 生成一个
     * @param traceId 全局跟踪 ID
     */
    public void addTraceId(String traceId) {
        if (StringUtils.isEmpty(traceId)) {
            MDC.put(TRACE_ID, UUID.randomUUID().toString().replace("-", ""));
        } else {
            MDC.put(TRACE_ID, traceId);
        }
    }

    public String getTraceId() {
        return MDC.get(TRACE_ID);
    }

    public void clear() {
        MDC.clear();
    }

}
```

三、在合适的设置上下文，例如 traceId：

- 在 http 接口的拦截器
- 在定时任务执行之前

![TraceIdInterceptor](/img/java/logging/logback/TraceIdInterceptor.png)

四、配置 Appender，参考配置文件 Sample

五、验证 MongoDB 数据。

## 配置 Encoders & Layouts

Logback 提供的 `Encoder` 和 `Layout` 的默认实现如下：

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

### 个性化格式

基于个性化格式，例如输出 traceid。

## 配置文件 Sample

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

## 使用 API

# 参考

http://logback.qos.ch/manual/appenders.html

http://logback.qos.ch/manual/appenders.html#AsyncAppender

https://github.com/logstash/logstash-logback-encoder

《[Java 中打印日志的正确姿势](https://mp.weixin.qq.com/s/v3aF1zVwsP5g8KcuJFP4fQ)》

《[细说 Java 主流日志工具库](https://mp.weixin.qq.com/s/68QncZycXgWPWgYg71wZGQ)》