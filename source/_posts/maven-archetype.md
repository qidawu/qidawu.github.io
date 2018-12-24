---
title: Maven 骨架快速搭建项目
date: 2018-12-08 20:20:46
updated:
tags: Java
---

工作中经常需要为新开展的业务创建新工程，如果每次都重新搭建、或者拷贝老项目，这些重复工作会影响开发效率，也不利于维护（例如为新工程引入新组件、或升级配置文件等）。Maven 提供了 archetype 骨架功能，用于抽取这些重复的配置和代码，以模板的方式创建新项目。

# 创建 archetype 工程样例

首先，利用 Maven 内置的 `maven-archetype-archetype` 构件创建一个骨架工程样例：

```
mvn archetype:generate
  -DgroupId=[your project's group id]
  -DartifactId=[your project's artifact id]
  -DarchetypeArtifactId=maven-archetype-archetype
```

创建成功后，其目录结构如下：

```java
archetype
|-- pom.xml  // archetype pom
`-- src
    `-- main
        `-- resources
            |-- META-INF
            |   `-- maven
            |       `--archetype.xml  // archetype descriptor
            `-- archetype-resources  // prototype files
                |-- pom.xml  // prototype pom
                `-- src
                    |-- main
                    |   `-- java
                    |       `-- App.java
                    `-- test
                        `-- java
                            `-- AppTest.java
```

骨架由以下四个部分组成，各文件作用如下：

| 组成部分             | 组成部分         | 路径                                              | 描述                                                         |
| -------------------- | ---------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| archetype pom        | 骨架的 POM       | 根目录下的 `pom.xml`                              |                                                              |
| archetype descriptor | 骨架描述符文件   | `src/main/resources/META-INF/maven/archetype.xml` | 这个文件列出了包含在 archetype 中的所有文件并将这些文件分类，因此 archetype 生成机制才能正确的处理。 |
| prototype pom        | 新工程的原型 POM | `src/main/resources/archetype-resources/pom.xml`  | archetype 插件会直接复制这个 `pom.xml`，然后替换其中的占位符 `${artifactId}`、`${groupId}`、`${version}` |
| prototype files      | 新工程的原型文件 | `src/main/resources/archetype-resources/`         | archetype 插件会直接复制这些文件                             |

# 配置描述符 archetype.xml

然后，配置 archetype.xml，详见：[archetype descriptor](http://maven.apache.org/archetype/archetype-models/archetype-descriptor/archetype-descriptor.html)

# 配置新工程的 pom.xml

使用占位符 `${artifactId}`、`${groupId}`、`${version}`，这些变量都将在 `archetype:generate` 命令运行时被初始化：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>${groupId}</groupId>
  <artifactId>${artifactId}</artifactId>
  <version>${version}</version>
  <packaging>jar</packaging>
 
  <name>A custom project</name>
  <url>http://www.myorganization.org</url>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

# 配置新工程的相关文件

将新工程所需文件，全部拷贝到 `src/main/resources/archetype-resources/` 目录下。

# 安装 archetype

创建骨架并配置完毕，首先安装到本地仓库：

```
mvn install
```

# 运行 archetype 插件

尝试创建项目，选择想要使用的骨架，并为新工程指定 `groupId` 和 `artifactId`：

```
mvn archetype:generate                                  \
  -DarchetypeGroupId=<archetype-groupId>                \
  -DarchetypeArtifactId=<archetype-artifactId>          \
  -DarchetypeVersion=<archetype-version>                \
  -DgroupId=<my.groupid>                                \
  -DartifactId=<my-artifactId>
```

Maven Archetype Plugin（骨架插件）能够让用户从现有的模板（即骨架）中创建 Maven 项目，也能够从现有的项目中创建骨架。其流程如下：

![Maven Archetype Plugin](/img/java/maven/archetype-overview.png)

从上图可见，该插件提供了如下目标（即命令）：

* [`archetype:generate`](http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html) creates a Maven project from an archetype: asks the user to choose an archetype from the archetype catalog, and retrieves it from the remote repository. Once retrieved, it is processed to create a working Maven project.
* [`archetype:create-from-project`](http://maven.apache.org/archetype/maven-archetype-plugin/create-from-project-mojo.html) creates an archetype from an existing project.
* [`archetype:crawl`](http://maven.apache.org/archetype/maven-archetype-plugin/crawl-mojo.html) search a repository for archetypes and updates a catalog.

# 参考

https://maven.apache.org/guides/introduction/introduction-to-archetypes.html

https://maven.apache.org/guides/mini/guide-creating-archetypes.html

http://maven.apache.org/archetype/maven-archetype-plugin/