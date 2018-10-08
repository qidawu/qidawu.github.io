---
title: Maven 继承关系与聚合关系
date: 2018-04-21 22:41:57
updated:
tags: Java
---

除了依赖关系，继承和聚合关系也是项目中很常用的两种关系。

# 继承关系

## 基础知识

- 在子项目中，使用 `<parent>` 指定父项目；
- 项目能继承父项目的一些配置（属性，依赖等）；
- 一般用于多个项目共享配置。
- 特别注意 `relativePath` 元素。虽非必填，但可以用作 Maven 的一个指示符，以在搜索本地和远程仓库之前首先搜索此路径，以达到优先使用本地父项目的目的（本地开发过程中有时我们会修改本地父项目的配置，例如修改依赖版本号）。

配置如下：

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

## 依赖管理

`<dependencyManagement>` 用于在具有层级关系的项目间统一管理依赖的版本，一般用在父项目中，通过它来管理 jar 包的版本，让子项目中引用一个依赖而不用显式的指定版本号，以达到**依赖版本统一管理**的目的。

这种方法的好处是显而易见的。依赖的细节（如版本号）可以在一个中心位置集中设置，并传播到所有继承的 POM。

但由于现实中有可能是 N 个项目都继承同一个父项目，如果把它们的依赖全部放到父项目的 `<DependencyManagement>` 中管理，势必会导致父项目的依赖配置急剧膨胀，形成一个巨型 POM。更为关键的是，后续各项目组都有升级各自提供的依赖版本的需要，如果大家都去改这个公共的父项目，版本管理会变得很混乱。

解决办法是按项目组粒度对依赖进行分组管理，分而治之。将每组依赖抽取成像 `spring-boot-dependencies` 一样，并在父项目中 `<dependencyManagement>` 使用 `scope=import` 方式组合起来，这样父项目的 POM 就会十分干净且稳定，各依赖的版本管理也能转由各自项目组的 POM 专门负责，类似一个金字塔模型。

例如某个公司的标准父项目 POM 如下：

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>test</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>  <!-- 必须是 pom -->

    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot 1 -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>1.5.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <!-- 此处省略 N 组依赖 -->
            ......
        </dependencies>
    </dependencyManagement>
</project>
```

开发阶段为了测试 Spring Boot 2 的兼容性，该公司 child 项目新开 git 分支，修改 POM 如下：

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>test</groupId>
        <artifactId>parent</artifactId>
        <version>1.0</version>
    </parent>
    <artifactId>child</artifactId>
  
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
  
    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot 2 -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.0.1.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

上例 child 项目覆盖了父项目 `<dependencyManagement>` 的 Spring Boot 版本，当测试通过后， `<dependencyManagement>` 即可去掉，转而升级父项目 Spring Boot 的版本号即可，这样所有子项目都会统一升级版本。

# 聚合关系

- 在父项目中，使用 `<modules>` 指定子项目；
- 在父项目中进行构建，会同时构建全部子项目；
- 一般用于批量管理一个项目的多个模块；

配置如下：

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

# 例子

可以参考 Spring Boot 项目的例子。

# 参考

https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-build-systems.html