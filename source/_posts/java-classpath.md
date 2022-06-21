---
title: Java 类加载篇（二）Classpath 类路径总结
date: 2018-11-04 21:04:43
updated:
tags: Java
typora-root-url: ..
---

# 各个类加载器的类路径（Classpath）

Java Launcher（即 [`java`](https://docs.oracle.com/en/java/javase/11/tools/java.html) 命令）启动 Java 虚拟机时，各个类加载器按以下**顺序**及**类路径**加载类：

![Class Loader](/img/java/classloader/classloader_hierarchy.png)

三个类加载器使用的类路径，从以下系统属性中获取，可以通过 `System.getProperty(...)` 获取查看：

```java
// Bootstrap ClassLoader
String property1 = System.getProperty("sun.boot.class.path");
// Extension ClassLoader
String property2 = System.getProperty("java.ext.dirs");
// Application ClassLoader
String property3 = System.getProperty("java.class.path");
```

下面详细介绍各个类加载器：

## Bootstrap ClassLoader

负责加载构成 Java 平台的基础类（Bootstrap classes），位于 `$JAVA_HOME/jre/lib` 目录，包括 `rt.jar` 和其它几个重要 jar 文件中的类。这些基础类包括 Java 类库（[Java Class Library (JCL)](https://en.wikipedia.org/wiki/Java_Class_Library)）的公共类，以及此库可用的私有类。

几乎所有的 Java 类库（JCL） 都存储在一个名为“`rt.jar`”的 [Java archive (jar)](https://en.wikipedia.org/wiki/JAR_(file_format)) 归档文件中，该文件随 [JRE](https://en.wikipedia.org/wiki/Java_Runtime_Environment) 和 [JDK](https://en.wikipedia.org/wiki/Java_Development_Kit) 发行版一起提供。Java 类库（`rt.jar`）位于默认的 bootstrap classpath（`$JAVA_HOME/jre/lib`）下，不必出现在为应用程序声明的 [classpath](https://en.wikipedia.org/wiki/Classpath_(Java)) 中。JRE 会使用引导类加载器（bootstrap class loader）找到 JCL。

Java 9 的[模块系统](https://en.wikipedia.org/wiki/Java_Module_System)目前已将这个单块的 `rt.jar` jar 包拆分并模块化。

## Extension ClassLoader

负责加载扩展 Java 平台的扩展类（Extension classes）。位于 `$JAVA_HOME/jre/lib/ext` 扩展目录的每个 `.jar` 文件都被假定为扩展文件，并使用 [Java Extension Framework](https://docs.oracle.com/javase/8/docs/technotes/guides/extensions/index.html) 扩展机制加载。

`sun.misc.Launcher$ExtClassLoader` 执行过程中，`URLClassPath` 的值如下：

![ext_classpath](/img/java/classloader/ext_classpath.png)

例如，可以将 MySQL 厂商驱动程序 `mysql-connector-java` 放到该扩展目录中。

## Application ClassLoader

负责加载由开发人员和第三方定义的未利用 Java 扩展机制的用户类（User classes）。

https://en.wikipedia.org/wiki/Classpath

> **Classpath** is a parameter in the [Java Virtual Machine](https://en.wikipedia.org/wiki/Java_virtual_machine) or the [Java compiler](https://en.wikipedia.org/wiki/Java_compiler) that specifies the location of user-defined [classes](https://en.wikipedia.org/wiki/Class_(computer_science)) and [packages](https://en.wikipedia.org/wiki/Java_package). The parameter may be set either on the [command-line](https://en.wikipedia.org/wiki/Command_line_interface), or through an [environment variable](https://en.wikipedia.org/wiki/Environment_variable).

为了查找 User classes，Java Launcher（即 [`java`](https://docs.oracle.com/en/java/javase/11/tools/java.html) 命令）将引用 *User Classpath* —— 一个包含了用户定义的类文件的目录、jar 包和 zip 包列表。Java Launcher（即 [`java`](https://docs.oracle.com/en/java/javase/11/tools/java.html) 命令）将这个 *User Classpath* 字符串放到 `java.class.path` 系统属性中。该值的来源及优先级如下：

* 默认值“ `.`”，表示当前工作目录下的所有类文件（如果在 jar 包中，则位于其下）。

* `CLASSPATH` 环境变量，覆盖默认值。

  ```bash
  # 查看 CLASSPATH 环境变量
  echo $CLASSPATH
  
  # 设置 CLASSPATH 环境变量
  set CLASSPATH=
  ```

* `-cp` 或 `-classpath` 命令行选项，覆盖默认值以及 `CLASSPATH` 环境变量。

  > A semicolon (`;`) separated list of directories, JAR archives, and ZIP archives to search for class files.
  >
  > Specifying `classpath` overrides any setting of the `CLASSPATH` environment variable. If the `classpath` option isn’t used and `CLASSPATH` isn’t set, then the user class path consists of the current directory (`.`).

* 由 `-jar` 选项指定的 jar 包，它覆盖上述所有值。如果使用此选项，则所有用户类必须来自指定的 jar 包。

  > Executes a program encapsulated in a JAR file. The `jarfile` argument is the name of a JAR file with a manifest that contains a line in the form `Main-Class:classname` that defines the class with the `public static void main(String[] args)` method that serves as your application's starting point.
  >
  > When you use `-jar`, the specified JAR file is the source of all user classes, and **other class path settings are ignored**.
  >
  > If you’re using JAR files, then see [jar](https://docs.oracle.com/en/java/javase/11/tools/jar.html#GUID-51C11B76-D9F6-4BC2-A805-3C847E857867).

参考：Setting the Class Path on [Windows](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/classpath.html) or [Unix](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/classpath.html)



⭐️ 注意点 1：*User Classpath* 使用字符串格式指定，路径不要含有空格，否则转义为 `%20` 之后会报错，例如：

```
java.lang.RuntimeException: Cannot resolve classpath entry: java.lang.RuntimeException: Cannot resolve classpath entry: D:\myprogram\mybatis%20tool\mybatis-generator-gui-0.8.4\target\classes\lib\mysql-connector-java-5.1.38.jar
```

⭐️ 注意点 2：每个路径使用以下方式进行分隔：

* 在类 Unix 系统中，以冒号（`:`）分隔
* 在 Windows 系统中，以分号（`;`）分隔

⭐️ 注意点 3：类文件具有反映“类的完全限定名称（class's fully-qualified name）”的子路径名。例如 `com.mypackage.MyClass`：

* 如果类存储在 `/myclasses` 目录，则 `/myclasses` 必须在 *User Classpath* 中，并且类文件的完整路径必须为 `/myclasses/com/mypackage/MyClass.class`。
* 如果类存储在 `myclasses.jar` 中，则 `myclasses.jar` 必须在  *User Classpath*  中，并且类文件必须存储 `myclasses.jar/com/mypackage/MyClass.class`。

下面来看几个例子，总结如下：

![CLASSPATH 例子](/img/java/classloader/User_Classpath.png)

### Unpacked Classes

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

总结，设置 *Classpath* 的两种方式：

* 使用 `CLASSPATH` 环境变量
* 使用 `-cp` 或 `-classpath` 命令行选项

### JAR files

* 单个 jar 包：使用绝对路径指定具体某个 jar 包
* 多个 jar 包：使用绝对路径加上通配符 `*`

### META-INF/MANIFEST.MF

如果程序已经打成 jar 包，需要使用[清单文件](https://en.wikipedia.org/wiki/Manifest_file)指定入口类及 `CLASSPATH`，并使用 `java -jar` 命令启动。例如 Tomcat 的 `bootstrap.jar` 引导包：

```
......
Main-Class: org.apache.catalina.startup.Bootstrap
Class-Path: commons-daemon.jar
```

# 例子

## Spring Boot 配置文件

> Spring Boot will automatically find and load `application.properties` and `application.yaml` files from the following locations when your application starts:
>
> 1. The classpath root
> 2. The classpath `/config` package
>
> The list is ordered by precedence (with values from **lower items overriding earlier ones**).

参考：[External Application Properties](/posts/spring-boot-getting-started/#External-Application-Properties)

## IDEA 如何查找类？

首先，为 IDEA 平台配置上你所拥有的 JDK 及路径：

![Platform Settings SDKs](/img/java/idea/platform_sdks.png)

然后，为你的项目指定一个默认 SDK：

![Project SDK](/img/java/idea/project_sdk.png)

搞掂之后，IDEA 会为项目载入指定版本的 SDK，将基础目录  `jre/lib/` 的 Bootstrap classes 和扩展目录  `jre/lib/ext/` 的 Extension classes 加入 classpath：

![External Libraries](/img/java/idea/external_libraries.png)

# 参考

https://docs.oracle.com/en/java/javase/11/tools/java.html

https://docs.oracle.com/en/java/javase/11/tools/jar.html

```bash
# Lists the table of contents for the archive.
$ jar tf jar-file

# [Extracting the Contents of a JAR File](https://docs.oracle.com/javase/tutorial/deployment/jar/unpack.html)
$ jar xf jar-file [archived-file(s)]
```

Setting the Class Path on [Windows](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/classpath.html) or [Unix](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/classpath.html)

http://cr.openjdk.java.net/~mr/jigsaw/ea/module-summary.html

Wikipedia

- https://en.wikipedia.org/wiki/Classpath_(Java)
- https://en.wikipedia.org/wiki/Java_class_file
- https://en.wikipedia.org/wiki/Java_Class_Library

