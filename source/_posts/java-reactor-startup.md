---
title: Java 响应式编程系列（二）Reactor 入门总结
date: 2020-08-08 21:48:49
updated:
tags: Java
typora-root-url: ..
---

# 先决条件

Reactor Core 需要在 `Java 8` 及以上版本运行。

# 依赖安装

自 Reactor 3 开始（since `reactor-core 3.0.4`, with the `Aluminium` release train），Reactor 使用 BOM (Bill of Materials) 模型来管理依赖。BOM 将一组相关的、可以良好协作的构建（Maven Artifact）组合在一起，提供版本管理。避免构件间潜在的版本不兼容风险。

为项目引入该 BOM：

```XML
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.projectreactor</groupId>
                <artifactId>reactor-bom</artifactId>
                <version>${reactor.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

然后就可以为项目添加相关依赖。注意忽略版本号，以便由 BOM 统一管理版本号（除非你想覆盖 BOM 管理的版本号）：

```XML
    <dependencies>
        <!-- Reactor -->
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

这里 Reactor Core 传递依赖于 Reactive Stream，如下：

![reactor_dependencies](/img/java/reactive-stream/reactor/reactor_dependencies.png)

```XML
    <dependency>
      <groupId>org.reactivestreams</groupId>
      <artifactId>reactive-streams</artifactId>
      <version>1.0.2</version>
      <scope>compile</scope>
    </dependency>
```

# 参考

[Reactor 框架](https://projectreactor.io/)，实现 Reactive Streams 规范，并扩展大量特性