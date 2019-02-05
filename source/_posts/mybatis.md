---
title: MyBatis
date: 2019-02-02 18:43:34
updated:
tags: Java
---



# 配置

## xml

```xml
<!-- MyBatis Plus：com.baomidou.mybatisplus.spring.MybatisSqlSessionFactoryBean -->
<!-- 配置 MyBatis 的 sqlSessionFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean
">
    <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
</bean>

<!-- DAO接口所在包名，Spring 会自动查找其下的类 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.example.dao" />
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

## Java Config

```java
import javax.sql.DataSource;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.mapper.MapperScannerConfigurer;

public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource);
    factoryBean.setConfigLocation(configLocation);
    factoryBean.setMapperLocations(resource);
	return factoryBean;
}

public MapperScannerConfigurer mapperScannerConfigurer() {
    MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
    mapperScannerConfigurer.setBasePackage(basePackage);
	return mapperScannerConfigurer;
}
```

## Spring Boot

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>x.x.x</version>
</dependency>
```

http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

