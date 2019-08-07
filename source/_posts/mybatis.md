---
title: MyBatis 框架解析
date: 2019-03-24 18:43:34
updated:
tags: Java
---

MyBatis 在 Java 业务开发领域可以说是首选的持久化框架，本文主要总结下 MyBatis 的核心配置和源码解析。

# MyBatis 产品组成

MyBatis 常用的产品组成汇总如下：

![MyBatis Products](/img/mybatis/mybatis_products.png)

# MyBatis 核心总结

![MyBatis Core](/img/mybatis/mybatis_core.png)

## 高级结果映射

```XML
<!-- 一对多嵌套查询 -->
<resultMap id="ExtendResultMap" extends="BaseResultMap" type="...XxxQO">
    <collection property="extendList" columnPrefix="sub_task_" resultMap="...XxxMapper.BaseResultMap" ofType="...XxxPO" />
</resultMap>

<!-- 一对多嵌套结果映射。注意表连接查询字段务必使用 AS 别名，避免手工映射时 column 取错列 -->
<resultMap id="ExtendResultMap" extends="BaseResultMap" type="...XxxQO">
    <collection property="extendList" column="task_no" select="...XxxMapper.getByTaskNo" ofType="...XxxPO" />
</resultMap>
```

# MyBatis 核心架构

![](/img/mybatis/mybatis_core_architecture.png)

# Spring 整合

## 依赖安装

基础依赖安装：

```xml
<!-- MyBatis -->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

Spring 整合依赖安装：

```xml
<!-- MyBatis Spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>x.x.x</version>
</dependency>

<!-- MyBatis Spring Boot Starter -->
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>x.x.x</version>
</dependency>
```

## Bean 配置

MyBatis-Spring 中，`SqlSessionFactoryBean` 用于创建 `SqlSessionFactory`。可通过 Spring 的 XML 配置文件或 Java Config 配置该工厂 bean：

### XML

```xml
<!-- 配置 MyBatis 的 sqlSessionFactory。MyBatis Plus 使用 com.baomidou.mybatisplus.spring.MybatisSqlSessionFactoryBean -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
</bean>

<!-- DAO接口所在包名，Spring 会自动查找其下的类 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.example.dao" />
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

### Java Config

```java
import javax.sql.DataSource;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.mapper.MapperScannerConfigurer;

@Bean
public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource);
    factoryBean.setConfigLocation(configLocation);
    factoryBean.setMapperLocations(resource);
    return factoryBean;
}

@Bean
public MapperScannerConfigurer mapperScannerConfigurer() {
    MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
    mapperScannerConfigurer.setBasePackage(basePackage);
    return mapperScannerConfigurer;
}
```

### Spring Boot

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>x.x.x</version>
</dependency>
```

http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

# 参考

http://www.mybatis.org/

https://github.com/mybatis

《MyBatis 从入门到精通》

《MyBatis 技术内幕》