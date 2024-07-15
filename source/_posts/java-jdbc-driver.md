---
title: Java 数据持久化系列（一）JDBC Driver 驱动程序总结
date: 2018-02-03 21:56:21
updated:
tags: [Java, JDBC]
typora-root-url: ..
---

# 总览

![JDBC Driver](/img/java/jdbc/jdbc-driver.png)

# 什么是 JDBC？

JDBC 表示 Java Database Connectivity，是 Java SE（Java 标准版）的一部分。JDBC API 用于在 Java 应用程序中执行以下活动：

1. 连接到数据库
2. 执行查询并将语句更新到数据库
3. 检索从数据库收到的结果

![JDBC API](/img/java/jdbc/usage-of-jdbc.png)

# 为什么要使用 JDBC？

在 JDBC 之前，ODBC API 是用于连接和执行命令的数据库 API 标准。但是，ODBC API 是使用 C 语言编写的驱动程序，依赖于平台。这就是为什么 Java 定义了自己的 JDBC API，它使用的 JDBC 驱动程序，是用 Java 语言编写的，具有与平台无关的特性，支持跨平台部署，性能也较好。

# 什么是 JDBC 驱动程序？

由于 JDBC API 只是一套接口规范，因此要使用 JDBC API 操作数据库，首先需要选择合适的驱动程序：

## 驱动程序四种类型

有四种类型的 JDBC 驱动程序：

1. JDBC-ODBC bridge driver (~~In Java 8, the JDBC-ODBC Bridge has been removed.~~)
2. Native-API driver (partially java driver)
3. Network-Protocol driver (Middleware driver, fully java driver)
4. **Database-Protocol driver (Thin driver, fully java driver)**，目前最常用的驱动类型，日常开发中使用的驱动 jar 包基本都属于这种类型，通常由数据库厂商直接提供，例如 `mysql-connector-java`。驱动程序把 JDBC 调用直接转换为数据库特定的网络协议，因此性能更好。驱动程序纯 Java 实现，支持跨平台部署。

> 我们知道，ODBC几乎能在所有平台上连接几乎所有的数据库。为什么 Java 不使用 ODBC？
>
> 答案是：Java 可以使用 ODBC，但最好是以JDBC-ODBC桥的形式使用（Java连接总体分为Java直连和JDBC-ODBC桥两种形式）。
>
> 那为什么还需要 JDBC？
>
> 因为ODBC 不适合直接在 Java 中使用，因为它使用 C 语言接口。从Java 调用本地 C代码在安全性、实现、坚固性和程序的自动移植性方面都有许多缺点。从 ODBC C API 到 Java API 的字面翻译是不可取的。例如，Java 没有指针，而 ODBC 却对指针用得很广泛（包括很容易出错的指针"void *"）。
>
> 另外，ODBC 比较复杂，而JDBC 尽量保证简单功能的简便性，同时在必要时允许使用高级功能。如果使用ODBC，就必须手动地将 ODBC 驱动程序管理器和驱动程序安装在每台客户机上。如果完全用 Java 编写 JDBC 驱动程序则 JDBC代码在所有 Java 平台上（从网络计算机到大型机）都可以自 动安装、移植并保证安全性。
>
> 总之，JDBC 在很大程度上是借鉴了ODBC的，从他的基础上发展而来。JDBC 保留了 ODBC 的基本设计特征，因此，熟悉 ODBC 的程序员将发现 JDBC 很容易使用。它们之间最大的区别在于：JDBC 以 Java 风格与优点为基础并进行优化，因此更加易于使用。

各类型的优缺点详见：

https://en.wikipedia.org/wiki/JDBC_driver

https://www.javatpoint.com/jdbc-driver

https://blog.csdn.net/autfish/article/details/52170053

## 驱动程序厂商实现

如果选定使用推荐的第四种驱动程序类型，接下来需要下载对应厂商的驱动程序，目前提供这些[支持列表](https://www.oracle.com/technetwork/java/index-136695.html)。

### MySQL Connector/J

例如最常用的 MySQL 数据库，提供了 [MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/)（即 `mysql-connector-java`）。Maven 依赖配置如下：

```xml
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>x.x.x</version>
</dependency>
```

关于 MySQL 驱动程序的更多信息，详见：https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-overview.html

- 驱动程序/数据源类名
- 连接 URL 语法
- 配置属性
- JDBC API 实现说明
- Java，JDBC 和 MySQL 类型
- 使用字符集和 Unicode
- 各种连接方式（如 SSL 安全连接）
- MySQL 错误码与 JDBC SQLState 代码的映射关系
- ……

# 如何使用 JDBC？

## 配置 JDBC URL

JDBC URL 提供了一种标识数据源的方法，以便相应的驱动程序识别它并与之建立连接。

因此，首先需要先配置好 JDBC URL，以便驱动程序注册完毕之后通过 JDBC URL 与数据源建立连接。

JDBC URL 的标准语法如下：

```
protocol//[hosts][/database][?properties]
```

详见：https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html

例如：`jdbc:mysql://localhost:3306/test?useUnicode=true;characterEncoding=utf-8`

### 常用协议

常见的 JDBC URL 协议及对应 Driver Class 如下：

![](/img/java/jdbc/jdbc-url.png)

### 配置属性

MySQL Connector/J：

https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-configuration-properties.html

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-configuration-properties.html

#### useSSL

javax.net.ssl.SSLHandshakeException: No appropriate protocol (protocol is disabled or cipher suites are inappropriate)

```
Caught while disconnecting...

** BEGIN NESTED EXCEPTION ** 

javax.net.ssl.SSLException
MESSAGE: closing inbound before receiving peer's close_notify

STACKTRACE:

javax.net.ssl.SSLException: closing inbound before receiving peer's close_notify
	at sun.security.ssl.Alert.createSSLException(Alert.java:133)
	at sun.security.ssl.Alert.createSSLException(Alert.java:117)
	at sun.security.ssl.TransportContext.fatal(TransportContext.java:340)
	at sun.security.ssl.TransportContext.fatal(TransportContext.java:296)
	at sun.security.ssl.TransportContext.fatal(TransportContext.java:287)
	at sun.security.ssl.SSLSocketImpl.shutdownInput(SSLSocketImpl.java:737)
	at sun.security.ssl.SSLSocketImpl.shutdownInput(SSLSocketImpl.java:716)
	at com.mysql.cj.protocol.a.NativeProtocol.quit(NativeProtocol.java:1319)
	at com.mysql.cj.NativeSession.quit(NativeSession.java:182)
	at com.mysql.cj.jdbc.ConnectionImpl.realClose(ConnectionImpl.java:1750)
	at com.mysql.cj.jdbc.ConnectionImpl.close(ConnectionImpl.java:720)
	at com.zaxxer.hikari.pool.PoolBase.quietlyCloseConnection(PoolBase.java:135)
	at com.zaxxer.hikari.pool.HikariPool.lambda$closeConnection$1(HikariPool.java:441)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)


** END NESTED EXCEPTION **
```

解决方法一：`useSSL=false`

解决方法二：注释掉 `java.security` 文件中的 `jdk.tls.disabledAlgorithms` 配置：

```
$ vim ~/.sdkman/candidates/java/8.0.362-zulu/zulu-8.jdk/Contents/Home/jre/lib/security/java.security

# jdk.tls.disabledAlgorithms=SSLv3, TLSv1, TLSv1.1, RC4, DES, MD5withRSA, \
#     DH keySize < 1024, EC keySize < 224, 3DES_EDE_CBC, anon, NULL, \
#     include jdk.disabled.namedCurves
```

#### connectionTimeZone

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-connp-props-datetime-types-processing.html#cj-conn-prop_connectionTimeZone

[Setting the MySQL JDBC Timezone Using Spring Boot Configuration](https://www.baeldung.com/mysql-jdbc-timezone-spring-boot)

#### useServerPrepStmts

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-connp-props-prepared-statements.html#cj-conn-prop_useServerPrepStmts

> Use server-side prepared statements if the server supports them?

MySQL 是否默认开启预编译，与 MySQL Server 的版本无关，而与 MySQL Connector/J（驱动程序）的版本有关，Connector/J 5.0.5 之前的版本默认开启预编译。Connector/J 5.0.5 及以后的版本默认不开启预编译，想启用 MySQL 预编译，就必须设置 `useServerPrepStmts=true`。

参考：《[JDBC 的 PreparedStatement 预编译详解](https://mp.weixin.qq.com/s/5JwMQJ-X2jBLfeTURLO68w)》

#### allowMultiQueries

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-connp-props-security.html#cj-conn-prop_allowMultiQueries

> Allow the use of `;` to delimit multiple queries during one statement (true/false). Default is `false`, and it does not affect the `addBatch()` and `executeBatch()` methods, which rely on `rewriteBatchedStatements` instead.

基于安全考虑，默认情况下，MySQL Connector/J 禁用 `;` 拼接 SQL。

如果想通过 MyBatis `foreach` 使用 `;` 拼接  `UPDATE`/`DELETE` 语句进行批量提交（但强烈不建议），需要设置 `useServerPrepStmts=true`。

#### rewriteBatchedStatements

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-connp-props-performance-extensions.html#cj-conn-prop_rewriteBatchedStatements

> Should the driver use multi-queries (regardless of the setting of `allowMultiQueries`) as well as rewriting of prepared statements for `INSERT` into **multi-value inserts** when `executeBatch()` is called? 
>
> Notice that this has the potential for [SQL injection](https://en.wikipedia.org/wiki/SQL_injection) if using plain `java.sql.Statement` and your code doesn't sanitize input correctly.
>
> Notice that for prepared statements, server-side prepared statements can not currently take advantage of this rewrite option, and that if you don't specify stream lengths when using `PreparedStatement.set*Stream()`, the driver won't be able to determine the optimum number of parameters per batch and you might receive an error from the driver that the resultant packet is too large. 
>
> `Statement.getGeneratedKeys()` for these rewritten statements only works when the entire batch includes `INSERT` statements. 
>
> Please be aware using `rewriteBatchedStatements=true` with `INSERT .. ON DUPLICATE KEY UPDATE` that for rewritten statement server returns only one value as sum of all affected (or found) rows in batch and it isn't possible to map it correctly to initial statements; in this case driver returns 0 as a result of each batch statement if total count was 0, and the `Statement.SUCCESS_NO_INFO` as a result of each batch statement if total count was > 0.

在 MySQL 中，`rewriteBatchedStatements` 的默认值取决于 MySQL 版本和 JDBC 驱动程序的配置。

* 在 MySQL 8.0 版本之前，`rewriteBatchedStatements` 的默认值是 `false`，这意味着批处理语句不会被重写优化。
* 从 MySQL 8.0 版本开始，`rewriteBatchedStatements` 的默认值被更改为 `true`，以提高默认情况下的性能。

要减少 JDBC 的网络调用次数改善性能，你可以设置 `rewriteBatchedStatements=true`，[并使用 `PreparedStatement` 的 `addBatch()` 方法并执行 `executeBatch()` 批量发送多个操作给数据库](/posts/java-jdbc-api/#批处理-1)。

根据执行的 DML 语句类型，使用不同的处理方法：

- 如果是 `INSERT` 语句，会整合成形如：`insert into t values (xx),(yy),(zz),...`
- 如果是 `UPDATE`/`DELETE` 语句，会整合成形如：`update t set … where id = 1; update t set … where id = 2; update t set … where id = 3; ...`

然后按 `maxAllowedPacket` 分批拼接 SQL 语句，然后按批次提交 MySQL。

参考：《[MySQL 批量操作](https://www.jianshu.com/p/04d3d235cb9f)》

#### maxAllowedPacket

> Maximum allowed packet size to send to server. If not set, the value of system variable `max_allowed_packet` will be used to initialize this upon connecting. This value will not take effect if set larger than the value of `max_allowed_packet`. Also, due to an internal dependency with the property "`blobSendChunkSize`", this setting has a minimum value of "8203" if "`useServerPrepStmts`" is set to "`true`".

参考：《[MySQL 批量插入数据，一次插入多少行数据效率最高？](https://blog.csdn.net/LJFPHP/article/details/99708888)》

## 注册驱动程序

有几种方式可以注册驱动程序，如下：

### 手工注册

```java
Class.forName("com.mysql.jdbc.Driver");  // 方式一，底层实现其实就是方式二
DriverManager.registerDriver(new com.mysql.jdbc.Driver());  // 方式二
System.setProperty("jdbc.drivers", "com.mysql.jdbc.Driver");  // 方式三
```

### 自动注册

从 JDBC API 4.0 开始，`java.sql.DriverManager` 类得到了增强，利用 **Java SPI 机制**从厂商驱动程序的 `META-INF/services/java.sql.Driver` 文件中自动加载 `java.sql.Driver` 实现类。 因此应用程序无需再显式调用 `Class.forName` 或 `DriverManager.registerDriver` 方法来注册或加载驱动程序。`java.sql.DriverManager` 源码分析如下，

```java
public class DriverManager {
    
    /**
     * Load the initial JDBC drivers by checking the System property
     * jdbc.properties and then use the {@code ServiceLoader} mechanism
     */
    static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }

    private static void loadInitialDrivers() {
        String drivers;
        try {
            drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
                public String run() {
                    return System.getProperty("jdbc.drivers");
                }
            });
        } catch (Exception ex) {
            drivers = null;
        }
        // If the driver is packaged as a Service Provider, load it.
        // Get all the drivers through the classloader
        // exposed as a java.sql.Driver.class service.
        // ServiceLoader.load() replaces the sun.misc.Providers()

        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {

                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();

                /* Load these drivers, so that they can be instantiated.
                 * It may be the case that the driver class may not be there
                 * i.e. there may be a packaged driver with the service class
                 * as implementation of java.sql.Driver but the actual class
                 * may be missing. In that case a java.util.ServiceConfigurationError
                 * will be thrown at runtime by the VM trying to locate
                 * and load the service.
                 *
                 * Adding a try catch block to catch those runtime errors
                 * if driver not available in classpath but it's
                 * packaged as service and that service is there in classpath.
                 */
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                // Do nothing
                }
                return null;
            }
        });

        println("DriverManager.initialize: jdbc.drivers = " + drivers);

        if (drivers == null || drivers.equals("")) {
            return;
        }
        String[] driversList = drivers.split(":");
        println("number of Drivers:" + driversList.length);
        for (String aDriver : driversList) {
            try {
                println("DriverManager.Initialize: loading " + aDriver);
                Class.forName(aDriver, true,
                        ClassLoader.getSystemClassLoader());
            } catch (Exception ex) {
                println("DriverManager.Initialize: load failed: " + ex);
            }
        }
    }

}
```

从上面源码中可见，当类加载器载入 `java.sql.DriverManager` 类时，会执行其静态代码块，从而执行 `loadInitialDrivers()` 方法。该方法实现中通过 Java SPI `ServiceLoader` 查找 classpath 下所有 jar 包内的 `META-INF/services` 目录，找到 `java.sql.Driver` 文件，加载其中定义的实现类并通过反射创建实例。以 `mysql-connector-java` 8.x 为例，该类定义就是 `com.mysql.cj.jdbc.Driver`，此时由于该 `Driver` 类内含静态代码块，会用 `new` 关键字创建自身实例并反向注册到 `DriverManager`，从而达到自动注册驱动程序的效果：

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {

    // Register ourselves with the DriverManager
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }

}
```

### 常见问题

如果有多个不同的驱动程序都被注册，调用 `DriverManager.getConnection` 方法通过 JDBC URL 获取数据源连接时，会使用第一个可用的驱动程序来创建连接。源码分析如下：

```java
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test?useUnicode=true;characterEncoding=utf-8")
```

`DriverManager` 会遍历已注册的驱动程序，尝试获取连接，关键代码：`Connection con = aDriver.driver.connect(url, info);`

```java
// DriverManager 源码

private static Connection getConnection(
        String url, java.util.Properties info, Class<?> caller) throws SQLException {

    ...
    
    for(DriverInfo aDriver : registeredDrivers) {
        // If the caller does not have permission to load the driver then
        // skip it.
        if(isDriverAllowed(aDriver.driver, callerCL)) {
            try {
                println("    trying " + aDriver.driver.getClass().getName());
                Connection con = aDriver.driver.connect(url, info);
                if (con != null) {
                    // Success!
                    println("getConnection returning " + aDriver.driver.getClass().getName());
                    return (con);
                }
            } catch (SQLException ex) {
                if (reason == null) {
                    reason = ex;
                }
            }

        } else {
            println("    skipping: " + aDriver.getClass().getName());
        }

    }
    
    ...
    
}
```

MySQL 驱动程序实现类会判断该 JDBC URL 是否支持：

```java
// com.mysql.cj.jdbc.NonRegisteringDriver 源码，实现 java.sql.Driver 接口

@Override
public java.sql.Connection connect(String url, Properties info) throws SQLException {

    try {
        if (!ConnectionUrl.acceptsUrl(url)) {
            /*
             * According to JDBC spec:
             * The driver should return "null" if it realizes it is the wrong kind of driver to connect to the given URL. This will be common, as when the
             * JDBC driver manager is asked to connect to a given URL it passes the URL to each loaded driver in turn.
             */
            return null;
        }

        ConnectionUrl conStr = ConnectionUrl.getConnectionUrlInstance(url, info);
        switch (conStr.getType()) {
            case SINGLE_CONNECTION:
                return com.mysql.cj.jdbc.ConnectionImpl.getInstance(conStr.getMainHost());

            case LOADBALANCE_CONNECTION:
                return LoadBalancedConnectionProxy.createProxyInstance((LoadbalanceConnectionUrl) conStr);

            case FAILOVER_CONNECTION:
                return FailoverConnectionProxy.createProxyInstance(conStr);

            case REPLICATION_CONNECTION:
                return ReplicationConnectionProxy.createProxyInstance((ReplicationConnectionUrl) conStr);

            default:
                return null;
        }

    } catch (UnsupportedConnectionStringException e) {
        // when Connector/J can't handle this connection string the Driver must return null
        return null;

    } catch (CJException ex) {
        throw ExceptionFactory.createException(UnableToConnectException.class,
                  Messages.getString("NonRegisteringDriver.17", new Object[] { ex.toString() }), ex);
    }
}
```

scheme 支持列表：

```java
// com.mysql.cj.conf.Type 枚举源码

/**
 * The database URL type which is determined by the scheme section of the connection string.
 */
public enum Type {
    SINGLE_CONNECTION("jdbc:mysql:", HostsCardinality.SINGLE), //
    FAILOVER_CONNECTION("jdbc:mysql:", HostsCardinality.MULTIPLE), //
    LOADBALANCE_CONNECTION("jdbc:mysql:loadbalance:", HostsCardinality.ONE_OR_MORE), //
    REPLICATION_CONNECTION("jdbc:mysql:replication:", HostsCardinality.ONE_OR_MORE), //
    XDEVAPI_SESSION("mysqlx:", HostsCardinality.ONE_OR_MORE);

    private String scheme;
    private HostsCardinality cardinality;
    
    ...
}
```

如果该 JDBC URL 没有对应可用的驱动程序，程序将抛出异常：`java.sql.SQLException: No suitable driver found for jdbc:...`。

# 参考

https://en.wikipedia.org/wiki/JDBC_driver

https://www.javatpoint.com/jdbc-driver

https://blog.csdn.net/autfish/article/details/52170053

https://dev.mysql.com/downloads/connector/j/

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-overview.html

《[MySQL 驱动 Bug 引发的事务不回滚问题](https://mp.weixin.qq.com/s/YjzgdkZOf0MYFIsWDoIMfA)》