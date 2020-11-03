---
title: Maven 实战系列（四）BOM 依赖管理（组合关系）
date: 2017-04-26 22:41:57
updated:
tags: Java
---

在大型项目中，BOM 用于将一组相关的、可以良好协作的构建（Maven Artifact）组合在一起，提供版本管理，避免构件间潜在的版本不兼容风险。

BOM 如下：

```XML
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>test</groupId>
    <artifactId>project-n-bom</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>  <!-- 必须是 pom -->
    
    <!-- 属性配置 -->
    <properties...>
    <!-- 依赖管理配置，声明该 BOM 管理的依赖 -->
    <dependencyManagement...>
</project>
```

例如常用的 Reactor、Spring Boot、Spring Cloud：

```XML
    <dependencyManagement>
        <dependencies>
            <!-- Import dependency management from Reactor -->
            <dependency>
                <groupId>io.projectreactor</groupId>
                <artifactId>reactor-bom</artifactId>
                <version>${reactor.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            
            <!-- Import dependency management from Spring Boot -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            
            <!-- Import dependency management from Spring Cloud -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

# 实践例子（一）

每个公司可能都拥有自己的标准父项目，为了解决 Maven 单继承问题，可以通过组合方式使用 `<dependencyManagement>` 依赖管理来享受其提供的版本管理的好处。

# 实践例子（二）

`<dependencyManagement>` 用于在具有层级关系的项目间统一管理依赖的版本，一般用在父项目中，通过它来管理 jar 包的版本，让子项目中引用一个依赖而不用显式的指定版本号，以达到**依赖版本统一管理**的目的。

这种方法的好处是显而易见的。依赖的细节（如版本号）可以在一个中心位置集中设置，并传播到所有继承的 POM。

但由于现实中有可能是 N 个项目都继承同一个父项目，如果把它们的依赖全部放到父项目的 `<DependencyManagement>` 中管理，势必会导致父项目的依赖配置急剧膨胀，形成一个巨型 POM。更为关键的是，后续各项目组都有升级各自提供的依赖版本的需要，如果大家都去改这个公共的父项目，版本管理会变得很混乱。

解决办法是按项目组粒度对依赖进行分组管理，分而治之。将每组依赖抽取成像 `spring-boot-dependencies` 一样的 BOM，并在父项目中 `<dependencyManagement>` 使用 `scope=import` 方式将这些 BOM 组合起来，这样父项目的 POM 就会十分干净且稳定，各组依赖的版本管理也能转由各自项目组的 BOM 专门负责，类似一个金字塔模型。

```
                   +------------+
                   | parent-pom |
                   +------+-----+
                          |
        +-----------------|-----------------+
        |                 |                 |
+-------v-------+ +-------+-------+ +-------v-------+
| Project-A-BOM | | Project-B-BOM | | Project-N-BOM |
+---------------+ +-------+-------+ +---------------+
                          |
           +--------------|---------------+
           |              |               |
     +-----v------+ +-----v------+ +------v-----+
     | Artifact-A | | Artifact-B | | Artifact-N |
     +------------+ +------------+ +------------+

```

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

上例 child 项目覆盖了父项目 `<dependencyManagement>` 的 Spring Boot 版本，当兼容性测试通过后，子项目 `<dependencyManagement>` 即可去掉，转而升级父项目 Spring Boot 的版本号即可，这样所有子项目都会统一升级版本。

# 参考

https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-build-systems.html

https://projectreactor.io/docs/core/release/reference/#getting-started-understanding-bom