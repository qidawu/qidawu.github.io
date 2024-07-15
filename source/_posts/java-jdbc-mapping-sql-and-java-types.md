---
title: Java 数据持久化系列（三）JDBC SQL 和 Java 数据类型映射总结
date: 2018-02-08 14:25:06
updated:
tags: [Java, JDBC]
typora-root-url: ..
---

由于 SQL 中的数据类型和 Java 编程语言中的数据类型并不相同，因此需要使用某种机制在使用 Java 类型的应用程序和使用 SQL 类型的数据库之间传输数据。

# SQL 类型映射到 Java 类型

不同数据库产品支持的 SQL 类型之间存在显着差异。即使不同的数据库支持具有相同语义的 SQL 类型，它们也可能为这些类型提供了不同的名称。例如，大多数主要数据库都支持 `large binary` 这种 SQL 类型，但是：

- MySQL 的命名为 `BINARY`、`VARBINARY`（详见：[The BINARY and VARBINARY Types](https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html)）
- Oracle 的命名为 `LONG RAW`
- Sybase 的命名为 `IMAGE`
- Informix 的命名为 `BYTE`
- DB2 的命名为 `LONG VARCHAR FOR BIT DATA`

幸运的是，JDBC 开发通常不需要关心目标数据库使用的实际 SQL 类型名称。大多数情况下，JDBC 开发将针对现有数据库表进行编程，并不需要关心用于创建这些表的确切 SQL 类型名称。

JDBC API 在 [`java.sql.Types`](https://docs.oracle.com/javase/8/docs/api/java/sql/Types.html) 类中定义了一组通用 SQL 类型标识符，旨在表达最常用的 SQL 类型。在使用 JDBC API 进行编程时，程序员通常可以使用这些 JDBC 类型来引用通用 SQL 类型，而无需关心目标数据库使用的确切 SQL 类型名称。

# 数据访问 API

为了在数据库和 Java 应用程序之间传输数据，JDBC API 提供了三组方法：

* `PreparedStatement` 类提供的用于将 Java 类型作为 SQL 语句参数发送的方法；
* `ResultSet` 类提供的用于将 `SELECT` 检索结果转换为 Java 类型的方法；
* `CallableStatement`类提供的用于将 `OUT` 参数转换为 Java 类型的方法。

## 静态数据访问

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

## 动态数据访问

在大多数情况下，用户都希望访问在编译期数据类型已知的结果或参数。但是某些情况下，应用程序在编译期无法获知它们访问的目标数据库的 schema。因此，除了静态的数据类型访问之外，JDBC 还提供了对动态的数据类型访问的支持。

访问在编译期数据类型未知的值，可以使用所有 Java 对象的共同父类 `Object` 类型：

* `PreparedStatement` 接口：

  ```java
  void setObject(int parameterIndex, Object x) throws SQLException;
  void setObject(int parameterIndex, Object x, int targetSqlType) throws SQLException;
  ```

* `ResultSet` 接口：

  ```java
  Object getObject(int columnIndex) throws SQLException;
  Object getObject(String columnLabel) throws SQLException;
  
  Object getObject(int columnIndex, java.util.Map<String,Class<?>> map) throws SQLException;
  Object getObject(String columnLabel, java.util.Map<String,Class<?>> map) throws SQLException;
  
  // ... will convert from the SQL type of the column to the requested Java data type, if the conversion is supported. If the conversion is not supported or null is specified for the type, a `SQLException` is thrown.
  <T> T getObject(int columnIndex, Class<T> type) throws SQLException;
  <T> T getObject(String columnLabel, Class<T> type) throws SQLException;
  ```
  
  ⚠️ 特别注意：对于最后一组动态数据访问方法，参数二 `type` 的值要与 `ResultSetMetaData.GetColumnClassName()` 返回的类型相匹配，类型转换才能成功。否则抛出异常如下：
  
  ![java.sql.SQLException](/img/java/jdbc/java.sql.SQLException.png)

  例如 MyBatis Plus [`com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler`](https://github.com/baomidou/mybatis-plus/blob/master/mybatis-plus-core/src/main/java/com/baomidou/mybatisplus/core/handlers/MybatisEnumTypeHandler.java#L118) 就使用到了 `ResultSet#getObject` 方法，如果类型转换失败则报错如上。
  
  关于 MySQL 类型与 Java 类型的映射关系，参考：https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-type-conversions.html

示例代码如下：

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

`boolean`, `char`, `byte`, `short`, `int`, `long`, `float`, `double` 八种基本数据类型将返回其对应的包装类型，其它的则返回对应的类型。

## 设置 `NULL` 值

> Some databases need to know the value's type even if the value itself is `NULL`. For this reason, **for maximum portability**, it's the JDBC specification itself that requires the `java.sql.Types` to be specified:

* `PreparedStatement` 接口：

  ```java
  // Sets the designated parameter to SQL NULL.
  void setNull(int parameterIndex, int sqlType) throws SQLException;
  void setNull(int parameterIndex, int sqlType, String typeName) throws SQLException;
  ```

参考：[Is `JdbcType` necessary in a MyBatis mapper?](https://stackoverflow.com/questions/18645820/is-jdbctype-necessary-in-a-mybatis-mapper)

# 参考

https://docs.oracle.com/javase/6/docs/technotes/guides/jdbc/getstart/mapping.html

[`java.sql.Types`](https://docs.oracle.com/javase/8/docs/api/java/sql/Types.html)

* https://github.com/qidawu/java-api-test/blob/master/src/test/java/jdbc/DynamicTypeTest.java
* [Java Examples for `java.sql.Types`](https://www.javatips.net/api/java.sql.types)
* [`org.apache.ibatis.type.JdbcType`](https://github.com/mybatis/mybatis-3/blob/master/src/main/java/org/apache/ibatis/type/JdbcType.java)
* [`com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler`](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-core/src/main/java/com/baomidou/mybatisplus/core/handlers/MybatisEnumTypeHandler.java)

