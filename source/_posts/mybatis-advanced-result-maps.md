---
title: Java 数据持久化系列（七）MyBatis 高级结果映射
date: 2019-03-27 22:54:24
updated:
tags: Java
typora-root-url: ..
---

# 总结

高级结果映射常用的两种方式：

* 嵌套结果映射，即利用表连接语法进行一次性的嵌套查询；
* 嵌套查询，即分开多次查询。注意 `association` 多对一关联存在 N+1 查询的性能问题，禁用。

以 `collection` 一对多关联为例，两种配置区别如下：

```XML
<!-- 一对多嵌套结果映射。注意表连接查询字段务必使用 AS 别名，避免手工映射时 column 取错列 -->
<resultMap id="ExtendResultMap" extends="BaseResultMap" type="...XxxQO">
    <collection property="extendList" columnPrefix="sub_task_" resultMap="...XxxMapper.BaseResultMap" ofType="...XxxPO" />
</resultMap>

<!-- 一对多嵌套查询，column 为嵌套查询的参数列，即“一”方的列 -->
<resultMap id="ExtendResultMap" extends="BaseResultMap" type="...XxxQO">
    <collection property="extendList" column="task_no" select="...XxxMapper.getByTaskNo" ofType="...XxxPO" />
</resultMap>
```

# 例子

下面提供一个例子，涉及的关联关系如下：学生多对一关联学校、一对多关联书本：

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
    <!-- 使用嵌套结果映射，即利用表连接语法进行一次性的嵌套查询。resultMap 配置见 SchoolMapper.xml -->
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