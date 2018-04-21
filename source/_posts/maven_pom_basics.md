---
title: Maven 基础知识
date: 2018-04-18 22:34:21
updated:
tags: Java
---

# Maven 坐标

Maven 定义了这样一组规则：世界上任何一个*构件*都可以使用 Maven 坐标唯一标识，Maven 坐标的元素包括：

| 坐标元素       | 描述                                       |
| ---------- | ---------------------------------------- |
| groupId    | 必选。定义当前 Maven 项目隶属的实际项目。                 |
| artifactId | 必选。定义实际项目中的一个 Maven 项目（模块），推荐使用实际项目名称作为前缀。 |
| version    | 必选。定义当前 Maven 项目所处版本。                    |
| packaging  | 可选。定义 Maven 项目的打包方式，默认为 `jar`。           |
| classifier | 不能直接定义。用来帮助定义构建输出的一些附属构件。如 javadoc、sources。 |

# POM 关系类型

Maven 一个强大的地方在于处理项目之间的关系。其中包括**依赖关系**（包括传递性依赖关系）、**继承关系**和**聚合关系**（多模块项目）。 

详见：《[Maven 依赖关系](/2018/04/20/maven-dependencies/)》、《[Maven 继承关系与聚合关系](/2018/04/21/maven-inheritance/)》

# 属性

Maven 属性是类似于 Ant 属性一样的值占位符。可以通过符号  `${X}`（其中 `X` 是属性）在 POM 中的任意位置访问它们的值。它们也可以被插件用作默认值使用，例如：

```xml
<project>
  ...
  <properties>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>
  ...
</project>
```

共有五种风格的占位符：

|                 |                                          |
| --------------- | ---------------------------------------- |
| `${env.X}`      | 返回 shell 的环境变量。例如 `${env.PATH}` 返回 PATH 环境变量。为了可靠性，Maven 2.1.0 开始将环境变量的名称归一化为全部大写。 |
| `${project.x}`  | 返回 pom.xml 文件中对应元素的值。 例如可通过 `${project.version}` 访问 `<project><version>1.0</version></project>`。 |
| `${settings.x}` | 返回 settings.xml 文件中对应元素的值。例如 `<settings><offline>false</offline></settings>` 可通过 `${settings.offline}` 访问。 |
| `${java.x}`     | 返回 Java 系统属性。 通过 `java.lang.System.getProperties()` 可访问的所有属性都可用作 POM 属性，例如`${java.home}`。 |
| `${x}`          | 返回 pom.xml 文件中 `<properties/>` 元素内的值。例如 `<spring.version>1.0.1-SNAPSHOT</spring.version>` 可通过 `${spring.version}` 访问。 |

# 参考

http://maven.apache.org/pom.html
