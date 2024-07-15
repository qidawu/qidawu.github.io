---
title: Java 数据持久化系列（七）MyBatis 批量插入与更新
date: 2018-03-29 23:43:34
updated:
tags: [Java, JDBC]
typora-root-url: ..
---

「传统的 OLTP 应用程序」和「现代的 WEB 应用程序」通常会执行许多小型数据更改操作，其中并发性能至关重要。本文记录如何优化这些批量插入与更新操作。

# 优化思路

https://dev.mysql.com/doc/refman/8.0/en/data-change-optimization.html

## INSERT

> If you are inserting many rows from the same client at the same time, use [`INSERT`](https://dev.mysql.com/doc/refman/8.0/en/insert.html) statements with multiple `VALUES` lists to insert several rows at a time. This is considerably faster (many times faster in some cases) than using separate single-row [`INSERT`](https://dev.mysql.com/doc/refman/8.0/en/insert.html) statements.

## UPDATE



# JDBC

[使用 `PreparedStatement` 的 `addBatch()` 方法并执行 `executeBatch()` 批量发送多个操作给数据库](http://localhost:4000/posts/java-jdbc-api/#批处理-1)，减少网络交互次数。

# MyBatis

## INSERT

[MyBatis 批量插入几千条数据慎用 `foreach`](https://blog.csdn.net/huanghanqian/article/details/83177178)。应使用 `BATCH` 模式，只发出一条 SQL 语句，预编译一次，但设置参数 N 次，执行一次。

```java
try (SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH)) {
  ...
}
```

![ExecutorType.BATCH](/img/java/mybatis/ExecutorType_BATCH.png)

相比之下，默认的 `SIMPLE` 模式，每一条语句都会发出一条 SQL，进行一次预编译，设置一次参数，执行一次。

![ExecutorType.SIMPLE](/img/java/mybatis/ExecutorType_SIMPLE.png)

两者性能相差很大。

参考：

* https://github.com/mybatis/mybatis-dynamic-sql
* https://mybatis.org/mybatis-dynamic-sql/docs/insert.html#batch-insert-support
* https://mybatis.org/mybatis-dynamic-sql/apidocs/org/mybatis/dynamic/sql/insert/render/BatchInsert.html

## UPDATE

# MyBatis Plus

## INSERT



## UPDATE

MyBatis Plus 只提供了根据主键 ID 进行批量更新的 `updateBatchById` 的方法，以下是根据某个或者多个非 ID 字段进行批量更新的自定义方法：

```java
    public boolean batchUpdate(Collection<SettlementDetailDO> entityList) {
        String sqlStatement = getSqlStatement(SqlMethod.UPDATE);
        return executeBatch(entityList, DEFAULT_BATCH_SIZE, (sqlSession, entity) -> {
            MapperMethod.ParamMap<Object> param = new MapperMethod.ParamMap<>();
            param.put(Constants.ENTITY, SettlementDetailDO.builder()
                    ...
                    .build());
            param.put(Constants.WRAPPER, Wrappers.<SettlementDetailDO>lambdaQuery()
                    .eq(SettlementDetailDO::getTransactionId, entity.getTransactionId())
                    .eq(SettlementDetailDO::getSettlementParty, entity.getSettlementParty())
                    .eq(SettlementDetailDO::getSettlementStatus, SettlementStatusEnum.INIT)
            );
            sqlSession.update(sqlStatement, param);
        });
```

参考：《[MyBatis Plus 根据某个指定字段批量更新数据库](https://www.panziye.com/java/4641.html)》

# 参考
