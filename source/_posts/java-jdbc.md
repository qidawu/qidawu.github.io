---
title: JDBC 总结
date: 2019-01-26 21:42:30
updated:
tags: Java
---

# 什么是 JDBC？

JDBC 是用于连接和执行数据库查询的 Java API。它是 JavaSE（Java 标准版）的一部分。我们可以在 Java 程序中使用 JDBC API 执行以下活动：

1. 连接到数据库
2. 执行查询并将语句更新到数据库
3. 检索从数据库收到的结果

![JDBC API](/img/java/jdbc/usage-of-jdbc.png)

# 为什么要使用 JDBC？

在 JDBC 之前，ODBC API 是用于连接和执行命令的数据库 API 标准。但是，ODBC API 是使用 C 语言编写的驱动程序，依赖于平台。这就是为什么 Java 定义了自己的 JDBC API，它使用的 JDBC 驱动程序，是用 Java 语言编写的，具有与平台无关的特性，支持跨平台部署，性能也较好。

# 什么是 JDBC 驱动程序？

由于 JDBC API 只是一套接口规范，因此要使用 JDBC API 操作数据库，首先需要选择合适的驱动程序：

## 驱动程序类型

JDBC API 使用 JDBC 驱动程序连接数据库。有四种类型的 JDBC 驱动程序：

1. JDBC-ODBC bridge driver (~~In Java 8, the JDBC-ODBC Bridge has been removed.~~)
2. Native-API driver (partially java driver)
3. Network-Protocol driver (Middleware driver, fully java driver)
4. **Database-Protocol driver (Thin driver, fully java driver)**，目前最常用的驱动类型，日常开发中使用的驱动 jar 包基本都属于这种类型，通常由数据库厂商直接提供，例如 `mysql-connector-java`。驱动程序把 JDBC 调用直接转换为数据库特定的网络协议，因此性能更好。驱动程序纯 Java 实现，支持跨平台部署。

各类型的优缺点详见：

https://en.wikipedia.org/wiki/JDBC_driver

https://www.javatpoint.com/jdbc-driver

https://blog.csdn.net/autfish/article/details/52170053

## 下载对应厂商的驱动程序

如果选定使用第四种驱动程序类型，接下来需要下载对应厂商的驱动程序，目前提供这些[支持列表](https://www.oracle.com/technetwork/java/index-136695.html)。

例如最常用的 MySQL 数据库，提供了 [MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/)（即 `mysql-connector-java`）。选择合适的版本，安装如下：

```xml
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>x.x.x</version>
</dependency>
```

# JDBC API 的使用步骤？

![JDBC 使用步骤](/img/java/jdbc/steps-to-connect-to-the-database-in-java.jpg)

# JDBC API 总结

## DriverManager 类

`DriverManager` 类充当用户和驱动程序之间的接口。它跟踪可用的驱动程序并处理数据库与相应驱动程序之间的连接。`DriverManager` 类维护了一个通过调用 `DriverManager.registerDriver()` 方法来注册自己的 `Driver` 类列表。

常用方法：

```java
static void registerDriver(Driver driver) // 用于通过 `DriverManager` 注册给定的驱动程序。
static void deregisterDriver(Driver driver)  // 用于从 `DriverManager` 取消注册给定的驱动程序（从列表中删除驱动程序）。
static Connection getConnection(String url)  // 用于与指定的 URL 建立连接。
static Connection getConnection(String url, String userName, String password)  // 用于与指定的 URL 建立连接，通过用户名和密码。
```

从 JDBC API 4.0 开始，`DriverManager.getConnection()` 方法得到了增强，可自动从驱动程序中的 `META-INF/services/java.sql.Driver` 文件中加载 JDBC 驱动程序。 因此，使用驱动程序 jar 库时，应用程序无需调用 `Class.forName` 方法来注册或加载驱动程序。

## Connection 接口

`Connection` 接口表示 Java 应用程序和数据库之间的会话（Session），它提供了许多事务管理方法如：

```java
void setAutoCommit(boolean status)  // 修改当前 `Connection` 对象的事务自动提交模式。默认为 `true`。
void setReadOnly(boolean readOnly)  // 修改当前 `Connection` 对象的只读状态以提示驱动程序开启数据库优化。
void setTransactionIsolation(int level)  // 修改当前 `Connection` 对象的事务隔离级别。
void commit()  // 保存自上次提交/回滚以来所做的所有更改。
void rollback()  // 丢弃自上次提交/回滚以来所做的所有更改。

// 在当前事务中设置或移除保存点。
Savepoint setSavepoint()
Savepoint setSavepoint(String name)
void releaseSavepoint(Savepoint savepoint)

void close()  // 关闭连接并立即释放 JDBC 资源。
```

`Connection` 接口同时也是一个工厂类，用于获取 `Statement`、`PreparedStatement` 和 `DatabaseMetaData` 对象：

```java
Statement createStatement(...)  // 创建一个可用于执行 SQL 查询或更新的语句对象。
PreparedStatement prepareStatement(...)  // 创建一个可用于执行 SQL 参数化查询或更新的语句对象。
CallableStatement prepareCall(...)  // 用于调用存储过程和函数。
DatabaseMetaData getMetaData()  // 用于获取数据库的元数据，例如数据库产品名称，数据库产品版本，驱动程序名称，表总数名称，总视图名称等。
```

## Statement 接口

`Statement` 接口提供用于执行数据库查询与更新的方法。`Statement`  接口是 `ResultSet` 的工厂，即它提供工厂方法来获取 `ResultSet` 的对象。

```java
ResultSet executeQuery(String sql)  // 用于执行 `SELECT` 查询并返回 `ResultSet` 的对象。
int executeUpdate(String sql)  // 用于执行指定的更新，如 `create`，`drop`，`insert`，`update`，`delete` 等。
boolean execute(String sql)  // 用于执行可能返回多种结果的查询。
```

除了通过上述方法来执行单个查询或更新，还可以通过下列方法执行批量命令：

```java
void addBatch(String sql)
void clearBatch()
int[] executeBatch()
```

使用批量命令前，记得先使用 `setAutoCommit()` 将事务的自动提交模式设置为 `false` 。

批处理允许您将相关的 SQL 语句分组到批处理中，并通过一次调用数据库来提交它们。当您一次性向数据库发送多个 SQL 语句时，可以减少通信开销，从而提高性能。参考：[JDBC - Batch Processing](https://www.tutorialspoint.com/jdbc/jdbc-batch-processing.htm)。

## PreparedStatement 接口

`PreparedStatement` 接口是 `Statement` 的子接口。它用于执行参数化查询（parameterized query），例如：

```sql
PreparedStatement stmt = connection.prepareStatement("insert into emp values(?, ?, ?)");
```

为什么要使用 `PreparedStatement`？

* **提升性能**：应用程序的性能会更快，因为 SQL 语句只会编译一次。
* **提升安全**

创建预编译的参数化查询语句后，需要通过下列方法设置对应参数：

```java
void setBoolean(int, boolean)
void setByte(int, byte)
void setShort(int short)
void setInt(int, int)
void setLong(int, long)
void setFloat(int, float)
void setDouble(int, double)
void setBigDecimal(int, BigDecimal)
void setString(int, String)
void setBytes(int, byte[])
void setDate(int, Date)
void setTime(int, Time)
void setTimestamp(int, Timestamp)
void setBinaryStream(int, InputStream)
void setCharacterStream(int, Reader)
void setRef(int, Ref)
void setBlob(int, Blob)
void setClob(int, Clob)
void setArray(int, Array)
...
```

参数设置完毕，就可以通过下列方法执行 SQL 语句：

```java
ResultSet executeQuery()  // 用于执行 `SELECT` 查询并返回 `ResultSet` 的对象。
int executeUpdate()  // 用于执行指定的更新，如 `create`，`drop`，`insert`，`update`，`delete` 等。
boolean execute()  // 用于执行可能返回多种结果的查询。
```

## ResultSet 接口

`ResultSet` 对象维护了一个指向 table 行的游标。游标初始值指向第 0 行。默认情况下，`ResultSet` 对象只能向前移动，并且不可更新。可以通过在 `createStatement(int, int)` 方法中传递指定参数修改该默认行为。

```java
boolean next()  // 将游标移动到当前位置的下一行。
boolean previous()  // 将游标移动到当前位置之前的一行。
boolean first()  // 将游标移动到结果集的第一行。
boolean last()  // 将游标移动到结果集的最后一行。
boolean absolute(int row)  // 将游标移动到结果集的指定行号。
boolean relative(int row)  // 将游标移动到结果集的相对行号，它可以是正数或负数。
```

将游标移动到指定行之后，可以通过以下方法获取当前行的指定列的数据：

```java
// 通过 int getInt(int columnIndex) 或 int getInt(String columnLabel)
Object getObject(int)
boolean getBoolean(int)
byte getByte(int)
short getShort(int)
int getInt(int)
long getLong(int)
float getFloat(int)
double getDouble(int)
BigDecimal getBigDecimal(int)
String getString(int)
byte[] getBytes(int)
Date getDate(int)
Time getTime(int)
Timestamp getTimestamp(int)
InputStream getBinaryStream(int)
Reader getCharacterStream(int)
Ref getRef(int)
Blob getBlob(int)
Clob getClob(int)
Array getArray(int)
...
```

此外，还可以直接获取 table 的元数据，例如列的总数，列名，列类型等：

```java
ResultSetMetaData getMetaData()
```

## ResultSetMetaData 接口

用于获取 table 的元数据，例如列的总数，列名，列类型等。

## DatabaseMetaData 接口

用于获取数据库的元数据，例如数据库产品名称，数据库产品版本，驱动程序名称，表总数名称，总视图名称等。

## RowSet 接口

`javax.sql.RowSet` 继承自 `java.sql.ResultSet`，是其包装器类。它包含类似 `ResultSet` 的表格数据，但使用起来非常简单灵活。其实现类如下：

![RowSet](/img/java/jdbc/rowset.jpg)

下面是一个不含事件处理代码的 `JdbcRowSet` 的简单示例：

```java
try (JdbcRowSet rowSet = RowSetProvider.newFactory().createJdbcRowSet()) {
    rowSet.setUrl(url);
    rowSet.setCommand("SELECT * FROM test");
    rowSet.execute();
    while (rowSet.next()) {
        int id = rowSet.getInt("id");
        String name = rowSet.getString("name");
        log.info("Result is {} {}", id, name);
    }
}
```

对比下面传统的 JDBC API，代码更加直观，需要直接管理的资源也更少：

```java
try (Connection conn = DriverManager.getConnection(url)) {
    try (Statement stmt = conn.createStatement()) {
        try (ResultSet rs = stmt.executeQuery("SELECT * FROM test")) {
            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                log.info("Result is {}, {}", id, name);
            }
        }
    }
}
```

要使用 `JdbcRowSet` 执行事件处理，需要在 `JdbcRowSet` 的 `addRowSetListener` 方法中添加 `RowSetListener` 的实例。`RowSetListener` 接口提供了必须实现的三个方法，如下：

```java
void cursorMoved(RowSetEvent event);
void rowChanged(RowSetEvent event);
void rowSetChanged(RowSetEvent event);
```

# 使用示例

## 存储图片

下例通过 `PreparedStatement` 接口的 `setBinaryStream()` 方法将图片（二进制信息）存储到数据库中。为了将图片存储到数据库中，需要在表中使用 `BLOB`（Binary Large Object）数据类型。

```java
try (Connection conn = DriverManager.getConnection(url)) {
    try (PreparedStatement stmt = conn.prepareStatement("INSERT INTO test(title, photo) VALUES(?, ?)")) {
        FileInputStream fileInputStream = new FileInputStream("F:\\test.jpg");
        stmt.setString(1, "pic1");
        stmt.setBinaryStream(2, fileInputStream);
        assertTrue(1 == stmt.executeUpdate());
    }
}
```

注意：这只是一个例子，生产环境中是不会将这类二进制信息存储到数据库中的，而是存储到专门的文件系统，以提升性能，并节省宝贵的数据库资源 :)

## 检索图片

```java
try (Connection conn = DriverManager.getConnection(url)) {
    try (PreparedStatement stmt = conn.prepareStatement("SELECT title, photo FROM test")) {
        try (ResultSet rs = stmt.executeQuery()) {
            while (rs.next()) {
                String title = rs.getString(1);
                log.info("title is {}", title);

                Blob photo = rs.getBlob(2);
                byte[] bytes = photo.getBytes(1, (int) photo.length());
                String fileName = String.format("F:\\%s.png", title);
                try (FileOutputStream fileOutputStream = new FileOutputStream(fileName)) {
                    fileOutputStream.write(bytes);
                }
            }
        }
    }
}
```

# 总结

最后，来总结下 JDBC 的几个要点：

![JDBC](/img/java/jdbc/jdbc.png)

# 参考

《[Getting Started with the JDBC API](https://docs.oracle.com/javase/6/docs/technotes/guides/jdbc/getstart/GettingStartedTOC.fm.html)》

https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/jdbc_41.html

https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/jdbc_42.html

https://docs.oracle.com/javase/9/docs/api/java/sql/package-summary.html

https://www.javatpoint.com/java-jdbc

https://www.tutorialspoint.com/jdbc/index.htm
