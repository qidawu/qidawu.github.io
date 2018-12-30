---
title: Java Launcher 如何查找类？
date: 2018-12-30 21:04:43
updated:
tags: Java
---

日常工作中掌握一些底层 Java Launcher 的类加载机制，有助于理解 Java 平台的运行过程从而解决一些 `ClassNotFound` 的问题。

# Java Launcher 如何查找类？

Java Launcher（即 `java`）启动 Java 虚拟机。虚拟机按以下顺序搜索并加载类：

## Bootstrap classes

构成 Java 平台的基础类，位于 `jre/lib/` 目录，包括 `rt.jar` 和其它几个重要 jar 文件中的类。这些基础类包括 Java 类库（[Java Class Library (JCL)](https://en.wikipedia.org/wiki/Java_Class_Library)）的公共类，以及此库可用的私有类。几乎所有的 JCL 都存储在一个名为“`rt.jar`”的 [Java archive (jar)](https://en.wikipedia.org/wiki/JAR_(file_format)) 归档文件中，该文件随 [JRE](https://en.wikipedia.org/wiki/Java_Runtime_Environment) 和 [JDK](https://en.wikipedia.org/wiki/Java_Development_Kit) 发行版一起提供。Java 类库（`rt.jar`）位于默认的 bootstrap classpath（`jre/lib/`）下，不必出现在为应用程序声明的 [classpath](https://en.wikipedia.org/wiki/Classpath_(Java)) 中。JRE 会使用引导类加载器（bootstrap class loader）找到 JCL。

## Extension classes

扩展 Java 平台的扩展类。位于 `jre/lib/ext/` 扩展目录的每个 `.jar` 文件都被假定为扩展文件，并使用 [Java Extension Framework](https://docs.oracle.com/javase/8/docs/technotes/guides/extensions/index.html) 扩展机制加载。

## User classes

由开发人员和第三方定义的未利用 Java 扩展机制的类。可以使用命令行上的 `-classpath` 选项（首选方法）或使用 `CLASSPATH` 环境变量来标识这些类的位置 。(See **Setting the Classpath** for [Windows](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/classpath.html) or [Unix](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/classpath.html).)

# Java Launcher 如何查找 User classes

为了查找 User classes，Java Launcher 启动程序将引用 *User Classpath* - 一个包含了用户定义的类文件的目录、jar 包和 zip 包列表。作为 Java 虚拟机或 Java 编译器中的一个参数，可以在命令行上或通过环境变量进行设置。

*User Classpath* 使用字符串格式指定，并用以下方式对列表中每个条目进行分隔：

* 在类 Unix 系统中，以冒号（`:`）分隔
* 在 Windows 系统中，以分号（`;`）分隔

Java Launcher 启动程序将这个 *User Classpath* 字符串放到 `java.class.path` 系统属性中。该值的来源及优先级如下：

* 默认值“ `.`”，表示当前工作目录下的所有类文件（如果在 jar 包中，则位于其下）。
* `CLASSPATH` 环境变量，覆盖默认值。查看方式：`echo $CLASSPATH`，设置方式：`set CLASSPATH=`
* `-cp` 或 `-classpath` 命令行选项，覆盖默认值以及 `CLASSPATH` 环境变量。
* 由 `-jar` 选项指定的 jar 包，它覆盖上述所有值。如果使用此选项，则所有用户类必须来自指定的 jar 包。

类文件具有反映“类的完全限定名称（class's fully-qualified name）”的子路径名。例如，如果类 `com.mypackage.MyClass` 存储在 `/myclasses` 目录，则 `/myclasses` 必须在 *User Classpath* 中，并且类文件的完整路径必须为 `/myclasses/com/mypackage/MyClass.class`。

如果类存储在名为 `myclasses.jar` 的 jar 包中，则 `myclasses.jar` 必须位于  *User Classpath*  中，并且类文件必须存储在 `myclasses.jar` 中的 `com/mypackage/MyClass.class`。

下面来看几个 `CLASSPATH` 的例子，总结如下：

![CLASSPATH 例子](/img/java/User_Classpath.png)

## class 文件

假设我们有一个名为主类：HelloWorld，存储在 *D:\myprogram* 目录下：

```
D:\myprogram\
      |
      ---> org\  
            |
            ---> mypackage\
                     |
                     ---> HelloWorld.class       
                     ---> SupportClass.class   
                     ---> UtilClass.class 
```

查看 Windows 下 `CLASSPATH` 环境变量：

```bash
$ echo $CLASSPATH
.;E:\Developer\Java\jdk1.8.0_191\lib;
```

由于 `CLASSPATH` 环境变量默认包含当前目录（`.`），这意味着当我们的工作目录为 `D:\myprogram\` 时，我们不需要显式指定 `CLASSPATH`：

```bash
$ cd /D/myprogram
$ java org.mypackage.HelloWorld
hello world
```

否则，需要使用 `-classpath` 参数显式指定如下：

```bash
$ java -classpath D:\myprogram org.mypackage.HelloWorld
hello world
```

## jar 包

* 单个 jar 包：使用绝对路径指定具体某个 jar 包

* 多个 jar 包：使用绝对路径加上通配符 `*`

## 在清单文件中设置路径

如果程序已经打成 jar 包，需要使用[清单文件](https://en.wikipedia.org/wiki/Manifest_file)指定入口类及 `CLASSPATH`，并使用 `java -jar` 命令启动。

# IDEA 如何查找类？

首先，为 IDEA 平台配置上你所拥有的 JDK：

![Platform Settings SDKs](/img/java/idea/platform_sdks.png)

然后，为你的项目指定一个默认 SDK：

![Project SDK](/img/java/idea/project_sdk.png)

搞掂之后，IDEA 会为项目载入指定版本的 SDK，将基础目录  `jre/lib/` 的 Bootstrap classes 和扩展目录  `jre/lib/ext/` 的 Extension classes 加入 classpath：

![External Libraries](/img/java/idea/external_libraries.png)

# 参考

https://en.wikipedia.org/wiki/Classpath_(Java)

https://en.wikipedia.org/wiki/Java_Class_Library

https://docs.oracle.com/javase/8/docs/technotes/tools/findingclasses.html