---
title: Java 数据持久化系列（七）MyBatis 使用总结
date: 2018-03-27 22:54:24
updated:
tags: [Java, JDBC]
typora-root-url: ..
---

![](/img/java/mybatis/mybatis_core.png)

# Configuration XML

## TypeHandler

https://mybatis.org/mybatis-3/configuration.html#typeHandlers

> Whenever MyBatis sets a parameter on a `PreparedStatement` or retrieves a value from a `ResultSet`, a `TypeHandler` is used to retrieve the value in a means appropriate to the Java type.

`org.apache.ibatis.type.TypeHandler` 接口实现类：

![TypeHandler](/img/java/mybatis/mybatis_api_TypeHandler.png)

[TypeHandler](https://mybatis.org/mybatis-3/configuration.html#typeHandlers) 可用于以下场景：

* 枚举字段处理
* 敏感字段脱敏、加解密处理
* [Binary-to-text encoding](https://en.wikipedia.org/wiki/Binary-to-text_encoding)
* ...

# Mapper XML Files

https://mybatis.org/mybatis-3/sqlmap-xml.html

## 查询

### 结果映射

https://mybatis.org/mybatis-3/sqlmap-xml.html#Result_Maps

结果映射，用于自定义结果集（Result Maps）：

![](/img/java/mybatis/mybatis_result_map.png)

参考源码：

[`DefaultResultSetHandler#handleRowValues`](https://mybatis.org/mybatis-3/apidocs/org/apache/ibatis/executor/resultset/DefaultResultSetHandler.html#handleRowValues-org.apache.ibatis.executor.resultset.ResultSetWrapper-org.apache.ibatis.mapping.ResultMap-org.apache.ibatis.session.ResultHandler-org.apache.ibatis.session.RowBounds-org.apache.ibatis.mapping.ResultMapping-)

[`DefaultResultSetHandler#createResultObject`](https://github.com/mybatis/mybatis-3/blob/mybatis-3.5.10/src/main/java/org/apache/ibatis/executor/resultset/DefaultResultSetHandler.java#L657)

#### 可变/不可变类的结果映射

如果是可变类（mutable classes），可以使用 Setter injection 进行结果映射（注意，必须指定 `property` 属性）：

> `id` – an ID result; flagging results as ID will help improve overall performance
> `result` – a normal result injected into a **field** or **JavaBean property**

```XML
  <resultMap id="BaseResultMap" type="...">
    <id column="id" jdbcType="BIGINT" property="id" />
    <result column="create_time" jdbcType="BIGINT" property="createTime" />
    <result column="update_time" jdbcType="BIGINT" property="updateTime" />
  </resultMap>
```

但如果是不可变类（immutable classes），由于都是 `final` field，没有 Setter 方法，则只能通过 Constructor injection 进行结果映射：

> `constructor` - used for injecting results into the constructor of a class upon instantiation
>
> - `idArg` - ID argument; flagging results as ID will help improve overall performance
> - `arg` - a normal result injected into the constructor

注意，`arg` 元素有两种设置方式：
* 有序的 `arg`。必须指定 `javaType` 属性，否则构造方法参数默认使用 `java.lang.Object` 类型会构造报错。

  ```XML
    <resultMap id="BaseResultMap" type="...">
      <constructor>
        <idArg column="id" javaType="java.lang.Long" />
        <arg column="create_time" javaType="java.lang.Long" />
        <arg column="update_time" javaType="java.lang.Long" />
      </constructor>
    </resultMap>
  ```

* 无序的 `arg`。通过指定 `name` 属性，可以忽略 `javaType` 属性。

  ```XML
    <resultMap id="BaseResultMap" type="...">
      <constructor>
        <idArg column="id" name="id" />
        <arg column="create_time" name="createTime" />
        <arg column="update_time" name="updateTime" />
      </constructor>
    </resultMap>
  ```

> When you are dealing with a `constructor` with many parameters, maintaining the order of arg elements is error-prone.
>
> Since 3.4.3, by specifying the `name` of each parameter, you can write `arg` elements in any order. To reference constructor parameters by their names, you can
>
> * either add `@Param` annotation to them
>
> * or compile the project with ['`-parameters`' compiler option](https://docs.oracle.com/en/java/javase/11/tools/javac.html) and enable `useActualParamName` (this option is enabled by default).
>
>   | 设置名               | 描述                                                         | 有效值           | 默认值 |
>   | :------------------- | :----------------------------------------------------------- | :--------------- | :----- |
>   | `useActualParamName` | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 `-parameters` 选项。 | `true`, `false` | `true` |
>   
>   ```XML
>   <!-- https://maven.apache.org/plugins/maven-compiler-plugin/examples/pass-compiler-arguments.html -->
>   <project>
>       <build>
>           <plugins>
>               <plugin>
>                   <groupId>org.apache.maven.plugins</groupId>
>                   <artifactId>maven-compiler-plugin</artifactId>
>                   <configuration>
>                       <compilerArgs>
>                           <arg>-parameters</arg>
>                       </compilerArgs>
>                   </configuration>
>               </plugin>
>           </plugins>
>       </build>
>   </project>
>   ```
>
> 
>
> `javaType` can be omitted if there is a property with the same name and type.

注意，如果抛出异常如下：

```
org.apache.ibatis.executor.ExecutorException: No constructor found in ...
```

解决方案：

* 如果是可变类（mutable classes），为类添加无参构造方法。
* 如果是不可变类（immutable classes），Mapper XML 结果映射 `resultMap` 必须使用正确参数的 `constructor`。

#### 嵌套结果映射

两种嵌套关系配置：

* `association` "has-one" type relationship
* `collection` "has many" type relationship

两种查询方式：

* 嵌套查询（Nested Select），即分开多次查询。
* 嵌套结果映射（Nested Results），即利用表连接语法进行一次性的连表查询；

> The `association` element deals with a "has-one" type relationship. For example, in our example, a `Blog` has one `Author`. An `association` mapping works mostly like any other result. You specify the target `property`, the `javaType` of the property (which MyBatis can figure out most of the time), the `jdbcType` if necessary and a `typeHandler` if you want to override the retrieval of the result values.
>
> Where the `association` differs is that you need to tell MyBatis how to load the association. MyBatis can do so in two different ways:
>
> - **Nested Select**: By executing another mapped SQL statement that returns the complex type desired.
> - **Nested Results**: By using nested result mappings to deal with repeating subsets of joined results.

总结：

|                          | 嵌套查询（Nested Select） | 嵌套结果映射（Nested Results） |
| ------------------------ | ------------------------- | ------------------------------ |
| `association` 多对一关联 | 禁用                      | 可用                           |
| `collection` 一对多关联  | 可用                      | 可用                           |

* `association` 多对一关联，嵌套查询（Nested Select）存在 N+1 次查询的性能问题，不建议使用。

  > While this approach **Nested Select** is simple, it will not perform well for large data sets or lists. This problem is known as the "N+1 Selects Problem". In a nutshell, the N+1 selects problem is caused like this:
  >
  > - You execute a single SQL statement to retrieve a list of records (the "+1").
  > - For each record returned, you execute a select statement to load details for each (the "N").
  >
  > This problem could result in hundreds or thousands of SQL statements to be executed. This is not always desirable.
  >
  > The upside is that MyBatis can lazy load such queries, thus you might be spared the cost of these statements all at once. However, if you load such a list and then immediately iterate through it to access the nested data, you will invoke all of the lazy loads, and thus performance could be very bad.
  >
  > And so, there is another way **Nested Results**.

* `collection` 一对多关联，两种配置方式的区别如下：

```XML
<!-- 一对多的嵌套结果映射（Nested Results）。注意表连接查询字段务必使用 AS 别名，避免手工映射时 column 取错列 -->
<resultMap id="ExtendResultMap" extends="BaseResultMap" type="...XxxQO">
    <collection property="extendList" columnPrefix="sub_task_" resultMap="...XxxMapper.BaseResultMap" ofType="...XxxPO" />
</resultMap>

<!-- 一对多的嵌套查询（Nested Select），column 为嵌套查询的参数列，即“一”方的列 -->
<resultMap id="ExtendResultMap" extends="BaseResultMap" type="...XxxQO">
    <collection property="extendList" column="task_no" select="...XxxMapper.getByTaskNo" ofType="...XxxPO" />
</resultMap>
```

下面演示一个例子，涉及的关联关系如下：学生多对一关联学校、一对多关联书本：

`StudentMapper.java`

```java
List<StudentQO> listStudents(int age);
```

`StudentQO.java`

```java
@Getter
@Setter
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class StudentQO extends StudentPO {

    /**
     * 多对一关联学校
     */
    private SchoolPO school;

    /**
     * 一对多关联书本
     */
    private List<BookPO> books;

}
```

`StudentMapper.xml`

```xml
<sql id="Base_column_list">
    T.id,
    ......
    T.create_time,
    T.update_time,
    T.version
</sql>

<sql id="Extend_column_list">
    <include refid="Base_column_list" />
    ,
    <!-- 见 SchoolMapper.xml -->
    <include refid="com.test.school.mapper.SchoolMapper.Base_Column_List" />
</sql>

<resultMap id="BaseResultMap" type="com.test.student.po.StudentPO">
    <id column="id" property="id" jdbcType="BIGINT"/>
    ......
    <result column="create_time" property="createTime" jdbcType="TIMESTAMP"/>
    <result column="update_time" property="updateTime" jdbcType="TIMESTAMP"/>
    <result column="version" property="version" jdbcType="TINYINT"/>
</resultMap>

<resultMap id="ExtendResultMap" extends="BaseResultMap" type="com.test.student.qo.StudentQO">
    <!-- 使用嵌套结果映射，即利用表连接语法进行一次性的连表查询。resultMap 配置见 SchoolMapper.xml -->
    <association property="school" columnPrefix="school_" resultMap="com.test.school.mapper.SchoolMapper.BaseResultMap" />
    <!-- 使用嵌套查询，即分开多次查询。column 为嵌套查询的参数列，select 引用见 BookMapper.xml -->
    <collection property="books" column="student_no" select="com.test.book.mapper.BookMapper.getByStudentNo" ofType="com.test.book.po.BookPO" />
</resultMap>

<!-- 使用表连接嵌套查询学生、学校 -->
<select id="listStudents" resultMap="ExtendResultMap">
    SELECT
        <include refid="Extend_column_list" />
    FROM t_student T
        INNER JOIN t_school E
        ON T.school_no = E.school_no
    WHERE T.age = #{age}
</select>
```

`SchoolMapper.xml`

```xml
<sql id="Base_Column_List">
    E.id AS school_id,
    ......
    E.create_time AS school_create_time,
    E.update_time AS school_update_time
</sql>

<resultMap id="BaseResultMap" type="com.test.school.po.SchoolPO">
    <id column="id" property="id" jdbcType="BIGINT"/>
    ......
    <result column="create_time" property="createTime" jdbcType="TIMESTAMP"/>
    <result column="update_time" property="updateTime" jdbcType="TIMESTAMP"/>
</resultMap>
```

### 自动结果映射

https://mybatis.org/mybatis-3/sqlmap-xml.html#Auto-mapping

相关[配置](https://mybatis.org/mybatis-3/zh/configuration.html)：

| 设置名                                                       | 描述                                                         | 有效值                       | 默认值    |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :--------------------------- | :-------- |
| `autoMappingBehavior`                                        | 指定 MyBatis 应如何自动映射列到字段或属性。<br>* `NONE` 表示关闭自动映射<br/>* `PARTIAL` 只会自动映射没有定义嵌套结果映射的字段<br/>* `FULL` 会自动映射任何复杂的结果集（无论是否嵌套） | `NONE`, `PARTIAL`, `FULL`    | `PARTIAL` |
| `autoMappingUnknownColumnBehavior`                           | 指定发现自动映射目标未知列（或未知属性类型）的行为。<br>* `NONE`: 不做任何反应<br>* `WARNING`: 输出警告日志（`'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior'` 的日志等级必须设置为 `WARN`）<br> * `FAILING`: 映射失败 (抛出 `SqlSessionException`) | `NONE`, `WARNING`, `FAILING` | `NONE`    |
| `mapUnderscoreToCamelCase`                                   | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | `true`, `false`              | `false`   |
| [`argNameBasedConstructorAutoMapping`](https://github.com/mybatis/mybatis-3/blob/mybatis-3.5.10/src/main/java/org/apache/ibatis/executor/resultset/DefaultResultSetHandler.java#L742) | 当应用构造器自动映射时，参数名称被用来搜索要映射的列，而不再依赖列的顺序。（新增于 3.5.10） | `true`, `false`              | `false`   |

### 缓存

#### 一级缓存

也称为本地缓存，与 `SqlSession` 绑定，只存在于 `SqlSession` 生命周期。

#### 二级缓存

https://github.com/mybatis/memcached-cache

https://github.com/mybatis/ignite-cache

https://github.com/mybatis/redis-cache

https://github.com/mybatis/hazelcast-cache

https://github.com/mybatis/caffeine-cache

https://github.com/mybatis/couchbase-cache

https://github.com/mybatis/oscache-cache

https://github.com/mybatis/ehcache-cache

参考：

《[MyBatis 二级缓存 关联刷新实现](https://mp.weixin.qq.com/s/c0BOu8wO7q6h6i0PU-K4Bg)》

## 插入

https://mybatis.org/mybatis-3/sqlmap-xml.html#insert_update_and_delete

### 回写主键

MyBatis 回写主键利用了 [JDBC 的特性](/posts/java-jdbc-api/#主键回写)，适用于支持自动生成主键的数据库，比如 MySQL 和 SQL Server。

> First, if your database supports auto-generated key fields (e.g. MySQL and SQL Server), then you can simply set `useGeneratedKeys="true"` and set the `keyProperty` to the target property and you're done.

```XML
<insert id="insertAuthor" useGeneratedKeys="true" keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```

`keyProperty` attribute:

>  (insert and update only) Identifies a property into which MyBatis will set the key value returned by `getGeneratedKeys`, or by a `selectKey` child element of the insert statement. Default: `unset`.

### 自定义生成主键

适用于不支持自动生成主键的数据库，比如 Oracle。

> MyBatis has another way to deal with key generation for databases that don't support auto-generated column types, or perhaps don't yet support the JDBC driver support for auto-generated keys.
>
> Here's a simple (silly) example that would generate a random ID (something you'd likely never do, but this demonstrates the flexibility and how MyBatis really doesn't mind).
>
> The `selectKey` statement would be run first, the `Author` id property would be set, and then the insert statement would be called. This gives you a similar behavior to an auto-generated key in your database without complicating your Java code.

```XML
<insert id="insertAuthor">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into Author
    (id, username, password, email,bio, favourite_section)
  values
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
</insert>
```

The `selectKey` element is described as follows:

```XML
<selectKey
  keyProperty="id"
  resultType="int"
  order="BEFORE"
  statementType="PREPARED">
```

注解方式：

```java
@Insert("insert into Author(id, username, password, email,bio, favourite_section) values (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})")
@SelectKey(keyProperty = "id", resultType = Long.class, before = false, statement = "SELECT LAST_INSERT_ID()")
```

# Dynamic SQL

https://mybatis.org/mybatis-3/dynamic-sql.html

## script

> For using dynamic SQL in annotated mapper class, `script` element can be used.

# 与 Spring 整合

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

## Spring Bean 配置

> In base MyBatis, the `SqlSessionFactory` is built using `SqlSessionFactoryBuilder`. In MyBatis-Spring, `SqlSessionFactoryBean` is used instead.

MyBatis-Spring 中，`SqlSessionFactoryBean` 用于创建 `SqlSessionFactory`。主要的配置参数如下：

* `dataSource` 数据源类
* `configLocation` 配置文件的位置
* `mapperLocations` Mapper XML 文件的位置
* `typeAliasesPackage`

MyBatis-Spring 中，Mapper API 的扫描查找有两种方式：

* 配置 `org.mybatis.spring.mapper.MapperScannerConfigurer` 主动扫描指定 `basePackage` 路径
* 为 Mapper API 加上类注解：`org.mybatis.spring.annotation.MapperScan`

下面演示如何通过 Spring 的 XML 配置文件或 Java Config 进行配置。

### XML

```xml
<!-- 配置 MyBatis 的 sqlSessionFactory。MyBatis Plus 使用 com.baomidou.mybatisplus.spring.MybatisSqlSessionFactoryBean -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
</bean>

<!-- DAO 接口所在包名，Spring 会自动查找其下的类 -->
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

@MapperScan(basePackages = {"..."})
public class MyBatisConfig {

    /**
     * 用于创建 SqlSessionFactory，以便获取 SqlSession
     **/
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setConfigLocation(configLocation);
        factoryBean.setMapperLocations(resource);
        return factoryBean;
    }

    /**
     * @MapperScan 或 MapperScannerConfigurer 二选一配置
     **/
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setBasePackage(basePackage);
        return mapperScannerConfigurer;
    }
    
}
```

## 事务

`SpringManagedTransactionFactory`

# 与 Spring Boot 整合

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>x.x.x</version>
</dependency>
```

http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

# 扩展

## MyBatis Generator

https://github.com/mybatis/generator

https://github.com/zouzg/mybatis-generator-gui

## MyBatis PageHelper

https://github.com/pagehelper/Mybatis-PageHelper

https://pagehelper.github.io/docs/howtouse/

## Mybatis Plus

https://baomidou.com/

https://github.com/baomidou/mybatis-plus

https://github.com/yulichang/mybatis-plus-join

https://www.jianshu.com/p/ceb1df475021

## IDEA Plugins

https://plugins.jetbrains.com/plugin/7293-mybatis-plugin

https://plugins.jetbrains.com/plugin/8321-free-mybatis-plugin