---
title: Maven 实战系列（三）继承与聚合关系
date: 2017-04-21 22:41:57
updated:
tags: Java
---

除了依赖关系，继承和聚合关系也是 Maven 项目中很常用的两种关系。

# 继承关系

- 在子项目中，使用 `<parent>` 指定父项目；
- 项目能继承父项目的一些配置（属性，依赖等）；
- 一般用于多项目共享父配置。
- 特别注意 `relativePath` 元素。虽非必填，但可以用作 Maven 的一个指示符，以在搜索本地和远程仓库之前首先搜索此路径，以达到优先使用本地父项目的目的（本地开发过程中有时我们会修改本地父项目的配置，例如修改依赖版本号）。

父项目配置如下：

```XML
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>test</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>  <!-- 必须是 pom -->

    <!-- 以下配置都可以被子项目继承 -->
    
    <!-- 属性配置 -->
    <properties...>
    <!-- 依赖管理配置，如 spring-cloud、spring-boot、reactor -->
    <dependencyManagement...>
    <!-- 依赖配置 -->
    <dependencies...>
    <!-- 构建配置 -->
    <build...>
    <!-- Nexus 分发包地址 -->
    <distributionManagement...>
    <!-- Nexus 仓库地址 -->
    <repositories...>
    <!-- Nexus 插件地址 -->
    <pluginRepositories...>
</project>
```

子项目配置如下：

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>test</groupId>
        <artifactId>parent</artifactId>
        <version>1.0</version>
        <relativePath>../parent</relativePath>
    </parent>

    <groupId>test</groupId>  <!-- 可继承自父项目 -->
    <artifactId>child-a</artifactId>
    <version>1.0</version>  <!-- 可继承自父项目 -->
</project>
```

# 聚合关系

- 在父项目中，使用 `<modules>` 指定子项目；
- 在父项目中进行构建，会同时构建全部子项目；
- 一般用于批量管理一个项目的多个模块；

父项目配置如下：

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
  
    <groupId>test</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>  <!-- 必须是 pom -->
  
    <modules>
        <!-- 子模块相对于父项目的路径，一般为 artifactId -->
        <module>child-a</module>
        <module>child-a</module>
    </modules>
</project>
```

