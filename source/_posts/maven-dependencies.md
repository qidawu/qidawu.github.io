---
title: Maven 实战系列（二）依赖关系总结
date: 2017-04-15 20:41:43
updated:
tags: [Java, Maven]
typora-root-url: ..
---

# 依赖关系

依赖关系是 Maven 项目之间最常见的关系。Maven 会自动解析所有项目的**直接依赖**和**传递性依赖**，并且根据规则正确判断**每个依赖范围（Dependency Scope）**。对于一些依赖冲突，自动进行**依赖调解（Dependency Mediation）**，或开发者**手动排除依赖（exclusions）**，以确保任何一个构件只有唯一的版本在依赖中存在。经过这些工作之后，得到的那些依赖被称为**已解析依赖（Resolved Dependency）**，从而产生一个 Effective POM。

## 依赖配置

下面是一段依赖配置：

```xml
<project>
    …
    <dependencies>
        <dependency>
            <groupId>…</groupId>        <!-- 依赖的构件属组 -->
            <artifactId>…</artifactId>  <!-- 依赖的构件ID -->
            <version>…</version>        <!-- 依赖的构件版本 -->
            <type>…</type>              <!-- 依赖类型，默认值为 jar -->
            <scope>…</scope>            <!-- 依赖范围 -->
            <optional>…</optional>      <!-- 可选依赖，默认情况下不会被继承，只有声明了该依赖才会继承 -->
            <exclusions>                <!-- 排除(传递性)依赖 -->
                <exclusion>…</exclusion>
                …
            </exclusions>
        </dependency>
        …
    </dependencies>
    …
</project>
```

## 依赖传递

Maven 会解析各个直接依赖的 POM，将那些必要的间接依赖，以**传递性依赖**的方式引入到当前项目中，而开发者不用再考虑该直接依赖还依赖了什么，或手动引入导致多余依赖：

![传递性依赖](/img/java/maven/transitive_dependencies.png)

## 依赖调解

Maven 调解依赖有两个基本原则：

1. 第一原则：**路径最短者优先**。
   假设 X 是 A 的传递性依赖：A->B->C->X(1.0) 依赖路径长度为 3，A->D->X(2.0) 长度为 2，则 X(2.0) 会先被解析使用。
2. 第二原则：**第一声明者优先**。
   在依赖路径长度相等的前提下，在 POM 中，依赖声明的顺序决定了谁会先被解析使用。

## 依赖范围 scope

依赖范围 `scope` 用于控制依赖与这三种 classpath（编译、测试、运行）的关系，即是否有效：

| 依赖范围（Scope） | 编译classpath | 测试classpath | 运行classpath | 例子                                                         |
| ----------------- | ------------- | ------------- | ------------- | ------------------------------------------------------------ |
| `compile`         | √             | √             | √             | spring-core                                                  |
| `test`            | ×             | √             | ×             | spring-test、junit、mockito                                  |
| `provided`        | √             | √             | ×             | servlet-api（运行时，由于容器如tomcat已经提供，无须重复引入） |
| `runtime`         | ×             | √             | √             | JDBC 驱动实现（编译时，只需 JDK 提供的 JDBC 接口；运行时，才需要接口的具体实现） |
| `system`          | √             | √             | ×             | 本地的，Maven 仓库之外的类库文件                             |
| `import`          | /             | /             | /             | Maven 2.0.9 版本后出的属性，`import` 只能在 `dependencyManagement` 的中使用，能解决 Maven 单继承问题，`import` 依赖关系实际上并不参与限制依赖关系的传递性。 |

## 可选依赖 optional

可选依赖 `optional` 为 `true` 时，该依赖不会被传递进来。

## 依赖排除 exclusions

当传递性依赖的结果不是自己预期的构件版本，可以排除依赖重新自定义。下例展示了使用 `<exclusions>` 排除 A 的传递性依赖 C，并显式声明依赖 C(1.10)：

```xml
<project>
    …
    <dependencies>
        <dependency>
            <groupId>…</groupId>
            <artifactId>A</artifactId>
            <version>1.0</version>
            <exclusions>  <!-- 排除 A 的传递性依赖 C -->
                <exclusion>
                    <groupId>…</groupId>
                    <artifactId>C</artifactId>
                </exclusion>
                …
            </exclusions>
        </dependency>

        <!-- 显式声明依赖 C(1.10) -->
        <dependency>
            <groupId>…</groupId>
            <artifactId>C</artifactId>
            <version>1.10</version>
        </dependency>
        …
    </dependencies>
    …
</project>
```

结果如图：

![依赖排除](/img/java/maven/exclusion_dependencies.png)

## 依赖归类

当多个构件使用同一版本号时，为了便于后期统一升级维护，可以使用 `<properties>` 属性统一定义：

```xml
<project>
    …
    <properties>
        <spring.version>5.0.0.RELEASE</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        ...
    </dependencies>
    …
</project>
```

# 常见问题

## 解决依赖冲突

例如在使用 Spring Boot 时，其依赖引入了一个有缺陷的 Jackson 框架版本 1.9.X，此时需要覆盖起步依赖引入的传递依赖：

方式一，如果项目未使用 Jackson 相关功能，可以使用 `<exclusions>` 排除依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <groupId>com.fasterxml.jackson.core</groupId>
    </exclusion>
  </exclusions>
</dependency>
```

方式二，如果项目使用了，可以利用“就近原则”，覆盖传递依赖的版本号，修复缺陷：

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.4.3</version>
</dependency>
```

## 依赖下载问题

有时候某个依赖下载有问题时，会导致本地环境找不到相关类。此时可以找到本地仓库下的相关 jar 包目录，排查下是否只有 `*.lastupdated` 文件而没有相应 jar 包。如果是，则删掉该目录重新下载依赖即可。

# 参考

http://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html

[Maven optional关键字透彻图解](https://juejin.im/post/6844903987322290189)