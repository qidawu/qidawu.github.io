---
title: Java 数据持久化系列（三）JDBC SQL 和 Java 数据类型映射总结
date: 2018-02-08 14:25:06
updated:
tags: [Java, JDBC]
typora-root-url: ..
---

# 映射概述

由于 SQL 中的数据类型和 Java 编程语言中的数据类型并不相同，因此需要使用某种机制在使用 Java 类型的应用程序和使用 SQL 类型的数据库之间传输数据。

# SQL 类型映射成 JDBC 类型

不同数据库产品支持的 SQL 类型之间存在显着差异。即使不同的数据库支持具有相同语义的 SQL 类型，它们也可能为这些类型提供了不同的名称。例如，大多数主要数据库都支持 large binary 这种 SQL 类型，但是：

- MySQL 的命名为 `BINARY`、`VARBINARY`（详见：[The BINARY and VARBINARY Types](https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html)）
- Oracle 的命名为 `LONG RAW`
- Sybase 的命名为 `IMAGE`
- Informix 的命名为 `BYTE`
- DB2 的命名为 `LONG VARCHAR FOR BIT DATA`

幸运的是，JDBC 开发通常不需要关心目标数据库使用的实际 SQL 类型名称。大多数情况下，JDBC 开发将针对现有数据库表进行编程，并不需要关心用于创建这些表的确切 SQL 类型名称。

JDBC API 在 `java.sql.Types` 类中定义了一组通用 SQL 类型标识符，旨在表达最常用的 SQL 类型。在使用 JDBC API 进行编程时，程序员通常可以使用这些 JDBC 类型来引用通用 SQL 类型，而无需关心目标数据库使用的确切 SQL 类型名称。



下表提供了 MySQL 类型与  JDBC 类型、Java 类型的映射关系：

![JDBC Types Mapped to Database-specific SQL Types](/img/java/jdbc/mysql-types.png)


## 基本 JDBC 类型

| Java 类型     | SQL 数据类型 |
| ---------------------- | ---------------------- |
| `byte[]` | `BINARY`、`VARBINARY`、`LONGVARBINARY` |
| `String` | `CHAR`，`VARCHAR`、`LONGVARCHAR` |
| `boolean` | `BIT` |
| `byte` | `TINYINT` |
| `short` | `SMALLINT` |
| `int` | `INTEGER` |
| `long` | `BIGINT` |
| `float` | `REAL` |
| `double` | `FLOAT`、`DOUBLE` |
| `java.math.BigDecimal` | `NUMERIC`、`DECIMAL` |
| `java.sql.Date`      | `DATE`              |
| `java.sql.Time`      | `TIME`              |
| `java.sql.Timestamp` | `TIMESTAMP`         |

## 高级 JDBC 类型

SQL 标准后续引入的数据类型，包括 `BLOB`， `CLOB`，`ARRAY`，`REF` 等等：

| Java 类型         | SQL 数据类型 | 备注              |
| ----------------- | ------------ | ----------------- |
| `java.sql.Array`  | `ARRAY`      | JDBC API 1.2 引入 |
| `java.sql.Blob`   | `BLOB`       | JDBC API 1.2 引入 |
| `java.sql.Clob`   | `CLOB`       | JDBC API 1.2 引入 |
| `java.sql.NClob`  | `NCLOB`      | JDBC API 1.6 引入 |
| `java.sql.Ref`    | `REF`        | JDBC API 1.2 引入 |
| `java.sql.RowId`  | `ROWID`      | JDBC API 1.6 引入 |
| `java.sql.Struct` | `STRUCT`     | JDBC API 1.2 引入 |
| `java.sql.SQLXML` | `XML`        | JDBC API 1.6 引入 |

# 数据访问 API

为了在数据库和 Java 应用程序之间传输数据，JDBC API 提供了三组方法：

* `PreparedStatement` 类提供的用于将 Java 类型作为 SQL 语句参数发送的方法；
* `ResultSet` 类提供的用于将 `SELECT` 检索结果转换为 Java 类型的方法；
* `CallableStatement`类提供的用于将 `OUT` 参数转换为 Java 类型的方法。

## 静态数据访问

### 标准映射

Java 程序从数据库中检索数据时，都必然会有某种形式的数据映射和数据转换。大多数情况下，JDBC 开发是知道目标数据库的 schema 的，例如表结构及其每列的数据类型。因此，JDBC 开发可以使用 `ResultSet`、`PreparedStatement`、`CallableStatement` 接口的强类型访问方法进行类型转换，如下：

* `PreparedStatement` 接口：

  ```java
  void setBoolean(int parameterIndex, boolean x) throws SQLException;
  void setByte(int parameterIndex, byte x) throws SQLException;
  void setInt(int parameterIndex, int x) throws SQLException;
  ...
  ```
  
* `ResultSet` 接口：

  ```java
  boolean getBoolean(int columnIndex) throws SQLException;
  boolean getBoolean(String columnLabel) throws SQLException;
  byte getByte(int columnIndex) throws SQLException;
  byte getByte(String columnLabel) throws SQLException;
  int getInt(int columnIndex) throws SQLException;
  int getInt(String columnLabel) throws SQLException;
  ...
  ```

### 自定义映射

自定义映射（SQL user-defined type (UDT) 到 Java 类）使用如下接口：

- `java.sql.SQLData` 接口
- `java.sql.SQLInput` 接口
- `java.sql.SQLOutput` 接口

## 动态数据访问

在大多数情况下，用户都希望访问在编译期数据类型已知的结果或参数。但是某些情况下，应用程序在编译期无法获知它们访问的目标数据库的 schema。因此，除了静态的数据类型访问之外，JDBC 还提供了对动态的数据类型访问的支持。

访问在编译期数据类型未知的值，可以使用所有 Java 对象的共同父类 `Object` 类型：

* `PreparedStatement` 接口：

  ```java
  void setObject(int parameterIndex, Object x, int targetSqlType)
  void setObject(int parameterIndex, Object x)
  ```

* `ResultSet` 接口：

  ```java
  Object getObject(int)
  ```

`boolean`, `char`, `byte`, `short`, `int`, `long`, `float`, `double` 八种基本数据类型将返回其对应的包装类型，其它的则返回对应的类型：

```java
try (Connection conn = DriverManager.getConnection(url)) {
    try (PreparedStatement stmt = conn.prepareStatement("SELECT * FROM test WHERE name = ?;")) {
        stmt.setObject(1, "李四", JDBCType.VARCHAR);
        try (ResultSet rs = stmt.executeQuery()) {
            while (rs.next()) {
                Object id = rs.getObject("id");
                String name = rs.getObject("name", String.class);
                log.info("Result is {}, {}", id instanceof Long, name);  // Result is true, 李四
            }
        }
    }
}
```

# 参考

https://docs.oracle.com/javase/6/docs/technotes/guides/jdbc/getstart/mapping.html

https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-type-conversions.html