---
title: Java 数据持久化系列（一）JDBC Driver 驱动程序总结
date: 2019-01-23 21:56:21
updated:
tags: Java
typora-root-url: ..
---

# 总览

![JDBC Driver](/img/java/jdbc/jdbc-driver.png)

# 什么是 JDBC？

JDBC 表示 Java Database Connectivity，是 JavaSE（Java 标准版）的一部分，。JDBC API 用于在 Java 应用程序中执行以下活动：

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

各类型的优缺点详见：

https://en.wikipedia.org/wiki/JDBC_driver

https://www.javatpoint.com/jdbc-driver

https://blog.csdn.net/autfish/article/details/52170053

## 驱动程序厂商实现

如果选定使用推荐的第四种驱动程序类型，接下来需要下载对应厂商的驱动程序，目前提供这些[支持列表](https://www.oracle.com/technetwork/java/index-136695.html)。

例如最常用的 MySQL 数据库，提供了 [MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/)（即 `mysql-connector-java`）。Maven 依赖配置如下：

```xml
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>x.x.x</version>
</dependency>
```

MySQL 驱动程序的更多信息，例如：

- 驱动程序/数据源类名
- 连接 URL 语法
- 配置属性
- JDBC API 实现说明
- Java，JDBC 和 MySQL 类型
- 使用字符集和 Unicode
- 各种连接方式（如 SSL 安全连接）
- MySQL 错误码与 JDBC SQLState 代码的映射关系
- ……

详见：https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-overview.html

# 注册驱动程序

有几种方式可以注册驱动程序，如下：

## 手工注册

```java
Class.forName("com.mysql.jdbc.Driver");  // 方式一，底层实现其实就是方式二
DriverManager.registerDriver(new com.mysql.jdbc.Driver());  // 方式二
System.setProperty("jdbc.drivers", "com.mysql.jdbc.Driver");  // 方式三
```

## 源码分析（自动注册）

从 JDBC API 4.0 开始，`DriverManager` 类得到了增强，利用 **Java SPI 机制**从厂商驱动程序的 `META-INF/services/java.sql.Driver` 文件中自动加载 `java.sql.Driver` 实现类。 因此应用程序无需再显式调用 `Class.forName` 或 `DriverManager.registerDriver` 方法来注册或加载驱动程序。`DriverManager` 源码分析如下：

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

从上面源码中可见，当类加载器载入 `DriverManager` 类时，会执行其 `static {}` 静态代码块，从而执行 `loadInitialDrivers()` 方法。该方法实现中调用了 `ServiceLoader.load(Driver.class)`，会加载 classpath 下所有 jar 包内的 `META-INF/services` 目录，找到 `java.sql.Driver` 文件，加载其中的类定义。以 `mysql-connector-java` 8.x 为例，该类定义就是 `com.mysql.cj.jdbc.Driver`，此时由于该 `Driver` 类内含静态代码块，会用 `new` 关键字创建自身实例并反向注册到 `DriverManager`，从而达到自动注册驱动程序的效果：

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

# JDBC URL

驱动程序注册完毕之后，接下来是通过 JDBC URL 与数据源建立连接。

JDBC URL 提供了一种标识数据源的方法，以便相应的驱动程序识别它并与之建立连接。

JDBC URL 的标准语法如下：

```
jdbc:<subprotocol>:<subname>
```

它有三个部分，用冒号分隔，分解如下：

* `jdbc` - 协议。JDBC URL 中的协议始终是 `jdbc`。
* `<subprotocol>` - 驱动程序的名称（如 `mysql`）或数据库连接机制的名称（如 `odbc`），可由一个或多个驱动程序支持。
* `<subname>` - 数据源的名称。

例如：

```
jdbc:mysql://localhost:3306/test?useUnicode=true;characterEncoding=utf-8
```

## 源码分析

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

## 常用列表

常见的 JDBC URL 前缀及对应 Driver Class 如下：

![](/img/java/jdbc/jdbc-url.png)

# 参考

https://en.wikipedia.org/wiki/JDBC_driver

https://www.javatpoint.com/jdbc-driver

https://blog.csdn.net/autfish/article/details/52170053

https://dev.mysql.com/downloads/connector/j/

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-overview.html