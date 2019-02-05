---
title: JDBC API 规范总结
date: 2019-01-26 21:42:30
updated:
tags: Java
---

# 总览

首先，来总览下 JDBC API：

![JDBC](/img/java/jdbc/jdbc-api.png)

# JDBC API 规范

JDBC API 作为 Java SE™（Java 标准版）的一部分，由以下部分组成：

* JDBC 核心 API —— `java.sql` package。

* JDBC 可选 API —— `javax.sql` package，是 Java EE™（Java 企业版）的重要组成部分。

其中，`java.sql` package 包含下列 API：

- 通过 `java.sql.DriverManager` 与数据库建立连接

  - `java.sql.DriverManager` 类 - 用于与驱动程序建立连接
  - `java.sql.SQLPermission` 类
  - `java.sql.Driver` 接口 - 提供用于注册和连接驱动程序的 API。
  - `java.sql.DriverPropertyInfo` 类 - 提供 JDBC 驱动程序的属性。

- 发送 SQL 语句到数据库

  - `java.sql.Connection` 接口 - 提供创建语句、管理连接及其属性的方法
  - `java.sql.Statement` 接口 - 用于发送基本的 SQL 语句
  - `java.sql.PreparedStatement` 接口 - 用于发送预编译语句或基本 SQL 语句（继承自`Statement`）
  - `java.sql.CallableStatement` 接口 - 用于调用数据库存储过程（继承自`PreparedStatement`）
  - `java.sql.Savepoint` 接口 - 在事务中提供保存点

- 检索和更新查询结果

  - `java.sql.ResultSet` 接口

- 标准映射（SQL 数据类型到 Java 类或接口）

- 自定义映射（SQL user-defined type (UDT) 到 Java 类）
- 元数据
  - `java.sql.DatabaseMetaData` 接口 - 提供有关数据库的信息
  - `java.sql.ResultSetMetaData` 接口 - 提供有关 `ResultSet` 对象的列信息
  - `java.sql.ParameterMetaData` 接口 - 提供有关 `PreparedStatement` 命令的参数信息
- 异常

  - `java.sql.SQLException` 类 - 被大多数方法抛出，当数据访问出现问题或出于其它原因
  - `java.sql.SQLWarning` 类 - 抛出表示警告
  - `java.sql.DataTruncation` 类 - 抛出表示数据可能已被截断
  - `java.sql.BatchUpdateException` 类 - 抛出表示批量更新中的部分命令未执行成功

下面重点看下常用的接口和类。

## DriverManager 类

`java.sql.DriverManager` 类充当用户和驱动程序之间的接口。它跟踪可用的驱动程序并处理数据库与相应驱动程序之间的连接。`DriverManager` 类维护了一个通过调用 `DriverManager.registerDriver()` 方法来注册自己的 `java.sql.Driver` 类列表。

常用方法：

```java
static void registerDriver(Driver driver) // 用于通过 `DriverManager` 注册给定的驱动程序。
static void deregisterDriver(Driver driver)  // 用于从 `DriverManager` 取消注册给定的驱动程序（从列表中删除驱动程序）。
static Connection getConnection(String url)  // 用于与指定的 URL 建立连接。
static Connection getConnection(String url, String userName, String password)  // 用于与指定的 URL 建立连接，通过用户名和密码。
```

关于 Driver 驱动程序注册，详见《[注册驱动程序](/2019/01/23/java-jdbc-driver/)》。

## Connection 接口

`java.sql.Connection` 接口表示 Java 应用程序和数据库之间的会话（Session），它提供了许多事务管理方法如：

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

`java.sql.Statement` 接口提供用于执行数据库查询与更新的方法。`Statement`  接口是 `ResultSet` 的工厂，即它提供工厂方法来获取 `ResultSet` 的对象。

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

`java.sql.PreparedStatement` 接口是 `java.sql.Statement` 的子接口。它用于执行参数化查询（parameterized query），例如：

```sql
PreparedStatement stmt = connection.prepareStatement("insert into emp values(?, ?, ?)");
```

为什么要使用 `PreparedStatement`？

* **提升性能**：应用程序的性能会更快，因为 SQL 语句只会编译一次。
* **提升安全**

创建预编译的参数化查询语句后，需要通过 `setXxx` 方法设置对应参数。参数设置完毕后，就可以通过下列方法执行 SQL 语句：

```java
ResultSet executeQuery()  // 用于执行 `SELECT` 查询并返回 `ResultSet` 的对象。
int executeUpdate()  // 用于执行指定的更新，如 `create`，`drop`，`insert`，`update`，`delete` 等。
boolean execute()  // 用于执行可能返回多种结果的查询。
```

## ResultSet 接口

`java.sql.ResultSet` 对象维护了一个指向 table 行的游标。游标初始值指向第 0 行。默认情况下，`ResultSet` 对象只能向前移动，并且不可更新。可以通过在 `createStatement(int, int)` 方法中传递指定参数修改该默认行为。

可以通过以下方法操作游标：

```java
boolean next()  // 将游标移动到当前位置的下一行。
boolean previous()  // 将游标移动到当前位置之前的一行。
boolean first()  // 将游标移动到结果集的第一行。
boolean last()  // 将游标移动到结果集的最后一行。
boolean absolute(int row)  // 将游标移动到结果集的指定行号。
boolean relative(int row)  // 将游标移动到结果集的相对行号，它可以是正数或负数。
```

将游标移动到指定行之后，可以通过 `getXxx` 方法获取当前行的指定列的数据。

此外，还可以直接获取 table 的元数据，例如列的总数，列名，列类型等：

```java
ResultSetMetaData getMetaData()
```

## ResultSetMetaData 接口

`java.sql.ResultSetMetaData` 用于获取 table 的元数据，例如列的总数，列名，列类型等。

## DatabaseMetaData 接口

`java.sql.DatabaseMetaData` 用于获取数据库的元数据，例如数据库产品名称，数据库产品版本，驱动程序名称，表总数名称，总视图名称等。

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

## DataSource 接口

# JDBC API 示例

JDBC API 的使用步骤如下：

![JDBC 使用步骤](/img/java/jdbc/steps-to-connect-to-the-database-in-java.jpg)

其中：
1. 步骤一：JDBC API 从 4.0 开始利用 Java SPI 机制自动加载驱动程序，可以省略该步骤。
2. 步骤二、三：如果使用如 Spring `JdbcTempate`、MyBatis 等框架，可以省略该步骤。
3. 步骤五：使用 `try-with-resources` 语句，可以省略该步骤。

下面来两个示例：

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

Using `try-with-resources` Statements to Automatically Close JDBC Resources: 

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

JDBC 4.1 (Java SE 7) introduces the ability to use a `try-with-resources` statement to automatically close `java.sql.Connection`, `java.sql.Statement`, and `java.sql.ResultSet` objects, regardless of whether a `SQLException` or any other exception has been thrown. See [The try-with-resources Statement](https://docs.oracle.com/javase/8/docs/technotes/guides/language/try-with-resources.html) for more information.

# 参考

《[Getting Started with the JDBC API](https://docs.oracle.com/javase/6/docs/technotes/guides/jdbc/getstart/GettingStartedTOC.fm.html)》

https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/jdbc_41.html

https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/jdbc_42.html

https://docs.oracle.com/javase/9/docs/api/java/sql/package-summary.html

https://www.javatpoint.com/java-jdbc

https://www.tutorialspoint.com/jdbc/index.htm

https://www.tutorialspoint.com/dbutils/index.htm