---
title: Java 类加载篇（一）类加载机制总结
date: 2018-11-04 21:04:43
updated:
tags: Java
typora-root-url: ..
---

# 类的加载过程

单个 .java 源文件的从加载到创建对象的过程如下：

![classloaoder_process](/img/java/classloader/classloaoder_process.jpg)

多个 .java 源文件经过 `javac` 编译后生成 .class 类文件（内含字节码），然后通过 `jar` 命令或其它构建工具（如 Maven、Gradle）打包生成可运行的 jar 包。最终通过 `java -jar` 命令运行 jar 包，执行其中清单文件（`META-INF/MANIFEST.MF`）中通过 `Main-Class` 指定的入口类的 `main` 方法以启动程序，并按照其 `Class-Path` 设置类路径。

从这里开始，就需要使用到类加载器将入口类（`Main-Class`）加载到 JVM。入口类在使用过程中如果使用到其它类，会根据类路径查找类文件并逐一加载。因此， jar 包中的类、及类路径中指定的类并不是一次性全部加载到 JVM 内存，而是使用到时才**动态加载**。可以指定启动参数 `-verbose:class` 输出类加载日志进行验证：

```java
public interface Flyable {
    void fly(String param);
}

public class Bird implements Flyable {
    @Override
    public void fly(String param) {}
}

public class Test {
    public static void main(String[] args) {
        System.out.println("===============");
        Bird bird = new Bird();
    }
}
```

类加载日志输出如下：

```
[Opened D:\tool\jdk1.8.0_131\jre\lib\rt.jar]
[Loaded java.lang.Object from D:\tool\jdk1.8.0_131\jre\lib\rt.jar]
[Loaded java.io.Serializable from D:\tool\jdk1.8.0_131\jre\lib\rt.jar]
[Loaded java.lang.Comparable from D:\tool\jdk1.8.0_131\jre\lib\rt.jar]
[Loaded java.lang.CharSequence from D:\tool\jdk1.8.0_131\jre\lib\rt.jar]
[Loaded java.lang.String from D:\tool\jdk1.8.0_131\jre\lib\rt.jar]
...
[Loaded Test from file:/D:/workspaces/project-test/target/classes/]
===============
[Loaded com.github.proxy.Flyable from file:/D:/workspaces/project-test/target/classes/]
[Loaded com.github.proxy.Bird from file:/D:/workspaces/project-test/target/classes/]
```

# 类的生命周期

类从加载到 JVM 内存到被从内存中释放，经历的生命周期如下：

![lifecycle_of_class](/img/java/classloader/lifecycle_of_class.png)

- 加载阶段：包括根据类或接口的二进制名称（[binary name](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#name)）查找其字节码文件（可能是之前由 `javac` 编译器源代码编译出的字节码文件；或者是通过动态编译，例如 JDK 动态代理使用的 `sun.misc.ProxyGenerator` 工具类编译出的字节码文件 `$Proxy0.class`），并构造成一个表示该类或接口的 `Class` 类对象。**加载阶段由类加载器 `ClassLoader` 及其子类负责实现：`findClass` 方法负责查找字节码文件，`defineClass` 方法负责构造成 `Class` 对象。**
- 验证阶段：确保类或接口的二进制代码在结构上是正确的。类文件校验器（Class File Verifier）会进行以下四类校验：
  - 文件完整性校验（File Integrity Check）：第一步也是最简单的一步是检查类文件的结构。 它确保类文件具有适当的签名（前四个字节为魔数 `0xCAFEBABE`），并且类文件中的每个结构都具有适当的长度。它检查类文件本身不能内容过长或过短，并且常量池仅包含有效条目。当然，类文件的长度可能有所不同，但是每个结构（例如常量池）的长度作为文件规范的一部分都包含其中。
  - 类完整性校验（Class Integrity Check）：
    - 该类具有父类（除非该类是 `Object`）。
    - 该父类不是一个 `final` 类，并且该子类不会尝试覆盖其父类中的 `final` 方法。
    - 常量池的条目格式正确，并且所有方法和字段引用均具有合法的名称和签名。
  - 字节码完整性校验（Bytecode Integrity Check）：执行字节码校验器（Bytecode Verifier），检查每个字节码以确定代码在运行时的实际行为，包括对方法参数和字节码操作数的数据流分析，堆栈检查和静态类型检查。是整个验证阶段中最复杂的一步。
  - 运行时完整性校验（Runtime Integrity Check）
- 准备阶段：包括为类或接口创建 `static` 静态字段（包括类变量和常量），并赋默认值。
- 解析阶段：包括检查符号引用是否正确、将符号引用替换为直接引用。
- 初始化阶段：
  - 类的初始化阶段包括执行 `static` 静态代码块、为 `static` 静态字段（变量）赋值。
  - 接口的初始化阶段包括为字段（接口字段默认为 `public static final` 常量）赋值。

各个步骤可以详见官方文档 ["Execution" chapter of The Java™ Language Specification](https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html)：

![execution](/img/java/classloader/execution.png)

# 类加载器源码解析

Java 虚拟机中的类加载器（`ClassLoader`）负责加载来自文件系统、网络或其它来源的类文件。`ClassLoader` 是一个抽象类，其继承结构如下：

![ClassLoader](/img/java/classloader/ClassLoader.png)

类加载后，每个 `Class` 对象都包含一个定义它的类加载器的[引用](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getClassLoader--)。可以通过以下方式查看：

```java
public class Test {
    @Test
    public void test() {
        // 结果为 null，因为启动类加载器为 C++ 编写
        ClassLoader bootstrapClassLoader = java.lang.String.class.getClassLoader();
        // sun.misc.Launcher$ExtClassLoader
        ClassLoader extClassLoader = com.sun.crypto.provider.DESKeyFactory.class.getClassLoader();
        // sun.misc.Launcher$AppClassLoader
        ClassLoader appClassLoader = Test.class.getClassLoader();
    }
}
```

`ClassLoader` 的核心方法如下：

## 类加载

### loadClass

`loadClass` 方法使用二进制名称（[binary name](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#name)）加载指定类。此方法的默认实现按以下顺序查找类：

1. 调用自身的 [`findLoadedClass(String)`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#findLoadedClass-java.lang.String-) 方法以检查是否已加载该类。
2. 递归调用父加载器的 [`loadClass`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#loadClass-java.lang.String-) 方法。 如果父加载器为 `null`，则使用虚拟机内置的启动类加载器（Bootstrap ClassLoader）。
3. 调用自身的 [`findClass(String)`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#findClass-java.lang.String-) 方法查找类。

如果上述步骤找到了类，并且 `resolve` 标记为 `true`，则此方法将在目标 `Class` 对象上调用 [`resolveClass(Class)`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#resolveClass-java.lang.Class-) 方法。

`ClassLoader` 整个过程的源码及注释如下：

```java
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查类是否已经被加载
            Class<?> c = findLoadedClass(name);
            // 未被加载的情况
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 如果父加载器不为 null，则委托父加载器去加载类
                    if (parent != null) {
                        // 调用父加载器的 loadClass 方法，委托其去加载类
                        c = parent.loadClass(name, false);
                    }
                    // 如果父加载器为 null，则委托 Bootstrap ClassLoader 去加载类
                    else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                // 如果父加载器都加载不到，则调用自身的 findClass 方法查找类
                if (c == null) {
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            // 如果上述步骤找到了类，并且 resolve 标记为 true，则在目标 Class 对象上调用 resolveClass(Class) 方法，进入“连接（Linking）”阶段（详见官方文档）
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

这整个加载过程被称为“双亲委派模型 (Delegation Model)”（流程见下图）。这种设计的好处体现在：

* 沙箱安全机制：例如自己写的 `java.lang.String` 类不会被加载，否则在 `defineClass` 方法这一步会报错，**防止恶意代码污染，核心 API 库被随意篡改。**核心 API 库只能由 `Bootstrap ClassLoader` 从`$JAVA_HOME/jre/lib` 目录进行加载。
* 避免类的重复加载：当父加载器已经加载了该类时，就没有必要再加载一次，**保证被加载类的唯一性。**

![classloader_hierarchy](/img/java/classloader/classloader_hierarchy.png)

这三个自带的 `ClassLoader` 细节详见类路径一节。

加载到的 `Class` 可以通过反射的方式实例化对象：

```java
Class<?> clazz = classLoader.loadClass("com.github.parent.HelloWorld");
Object instance = clazz.newInstance();
```

### findClass

实现“加载阶段（Loading）”的查找功能。该方法应当被子类覆盖重写，用于使用指定的二进制名称（[binary name](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#name)）查找类或接口的字节码文件。`ClassLoader` 的默认实现是抛出一个 `ClassNotFoundException`：

```java
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
```

可以通过覆盖这个方法实现一个自定义的类加载器（参考 [ClassLoader](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html) 介绍的例子，自定义 `NetworkClassLoader`）。

类可以按需动态加载到内存，这是 Java 的一大特点，也称为运行时绑定，或动态绑定。类文件的获取途径如下：

- 从 ZIP 包中读取，最常见，JAR，WAR，EAR 格式的基础。
- 从网络中获取，典型场景是 Applet。
- 运行时计算生成，典型情景是 JDK 动态代理技术。
- 从其它文件中生成，典型场景是 JSP 应用，即由 JSP 文件生成对应的 Servlet Class 类。

如果用户想从其它位置加载类文件，可以自定义类加载器，或者使用自带的 `URLClassLoader` 从本地路径（`file:/`）或网络路径（`http://`）加载类文件，如下：

```java
// 从 E 盘中加载类文件
URLClassLoader classLoader = new URLClassLoader(new URL[] {new URL("file:/e:/")});
// 从 localhost 中加载类文件
// URLClassLoader classLoader = new URLClassLoader(new URL[] {new URL("http://localhost/testfile/")});
Class<?> clazz = classLoader.loadClass("com.github.parent.HelloWorld");
Object instance = clazz.newInstance();
// java.net.URLClassLoader
String name = instance.getClass().getClassLoader().getClass().getName();
```

下面是子类 `URLClassLoader` 的默认实现，源码及注释如下：

```java
    protected Class<?> findClass(final String name)
        throws ClassNotFoundException
    {
        final Class<?> result;
        try {
            result = AccessController.doPrivileged(
                new PrivilegedExceptionAction<Class<?>>() {
                    public Class<?> run() throws ClassNotFoundException {
                        // 将二进制名称替换为文件路径（类似全限定名），例如：com.github.HelloWorld > com/github/HelloWorld.class
                        String path = name.replace('.', '/').concat(".class");
                        // 从 URLClassPath 对象中查找文件
                        Resource res = ucp.getResource(path, false);
                        if (res != null) {
                            try {
                                // 如果找到文件，则构造为 Class 类实例
                                return defineClass(name, res);
                            } catch (IOException e) {
                                throw new ClassNotFoundException(name, e);
                            }
                        } else {
                            return null;
                        }
                    }
                }, acc);
        } catch (java.security.PrivilegedActionException pae) {
            throw (ClassNotFoundException) pae.getException();
        }
        if (result == null) {
            throw new ClassNotFoundException(name);
        }
        return result;
    }
```

可见，如果找不到类（`result == null`），最底层的 `ClassLoader` 将抛出 `ClassNotFoundException`。

### defineClass

实现“加载阶段（Loading）”的构造功能。

`ClassLoader` 提供了四个 `defineClass` 方法可供自定义类加载器时使用，如下图。其中，第二个方法最常使用：`defineClass(String name, byte[] b, int off, int len)`。

![defineClass](/img/java/classloader/defineClass.png)

其调用的底层源码如下，会调用 `preDefineClass` 和 `postDefineClass` 进行预处理和后置处理：

```java
    protected final Class<?> defineClass(String name, byte[] b, int off, int len,
                                         ProtectionDomain protectionDomain)
        throws ClassFormatError
    {
        protectionDomain = preDefineClass(name, protectionDomain);
        String source = defineClassSourceLocation(protectionDomain);
        Class<?> c = defineClass1(name, b, off, len, protectionDomain, source);
        postDefineClass(c, protectionDomain);
        return c;
    }
```

如果自定义类加载器打破了双亲委派模型，然后还去加载核心 API 库，例如自己伪造一个 `java.lang.String`，会报错如下：

```
java.lang.SecurityException: Prohibited package name: java.lang
    at java.lang.ClassLoader.preDefineClass(ClassLoader.java:659)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:758)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
    at com.github.MyClassLoader.findClass(...)
    at com.github.MyClassLoader.loadClass(...)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
    ...
```

这是由于 `preDefineClass` 预处理方法进行了二进制名称的前缀校验，源码如下，关键判断 `name.startsWith("java.")` 抛出异常：

```java
    /* Determine protection domain, and check that:
        - not define java.* class,
        - signer of this class matches signers for the rest of the classes in
          package.
    */
    private ProtectionDomain preDefineClass(String name,
                                            ProtectionDomain pd)
    {
        if (!checkName(name))
            throw new NoClassDefFoundError("IllegalName: " + name);

        // Note:  Checking logic in java.lang.invoke.MemberName.checkForTypeAlias
        // relies on the fact that spoofing is impossible if a class has a name
        // of the form "java.*"
        // 关键判断
        if ((name != null) && name.startsWith("java.")) {
            throw new SecurityException
                ("Prohibited package name: " +
                 name.substring(0, name.lastIndexOf('.')));
        }
        if (pd == null) {
            pd = defaultDomain;
        }

        if (name != null) checkCerts(name, pd.getCodeSource());

        return pd;
    }
```

### resolveClass

进入类加载的“连接（Linking）”阶段（详见官方文档 "Execution" chapter of The Java™ Language Specification）。

## 资源加载

使用场景：例如 Spring Factories 机制中 `SpringFactoriesLoader` 加载类路径下的文件：

```java
classLoader.getResources("META-INF/spring.factories")
```

文件加载后，通过 key-value 的方式读取指定 key，并以反射的方式实例化指定的类型。

### getResource

查找指定名称的资源（图像、音频、文本等）。资源的名称是用“/”分隔的路径名，用于标识资源。
该方法首先递归调用父加载器查找资源；如果父加载器为 `null`，则使用虚拟机内置的启动类加载器（Bootstrap ClassLoader）。如果父加载器查找失败，则调用自身的 `findResource(String)` 查找资源。整个资源加载过程仍然为“双亲委派模型 (Delegation Model)”：

```java
    public URL getResource(String name) {
        URL url;
        if (parent != null) {
            url = parent.getResource(name);
        } else {
            url = getBootstrapResource(name);
        }
        if (url == null) {
            url = findResource(name);
        }
        return url;
    }
```

### findResource

查找指定名称的资源。类加载器实现应当重写此方法以指定在何处查找资源。默认返回 `null`：

```java
    protected URL findResource(String name) {
        return null;
    }
```

下面是子类 `URLClassLoader` 的默认实现：

```java
    public URL findResource(final String name) {
        /*
         * The same restriction to finding classes applies to resources
         */
        URL url = AccessController.doPrivileged(
            new PrivilegedAction<URL>() {
                public URL run() {
                    return ucp.findResource(name, true);
                }
            }, acc);

        return url != null ? ucp.checkURL(url) : null;
    }
```


# 类路径

Java Launcher（即使用 `java` 命令）启动 Java 虚拟机时，类加载器按以下顺序搜索指定路径并加载类：

![Class Loader](/img/java/classloader/default_classloader.png)

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

负责加载由开发人员和第三方定义的未利用 Java 扩展机制的类（User classes）。可以使用命令行上的 `-classpath` 选项（首选方法）或使用 `CLASSPATH` 环境变量来标识这些类的位置 。(See **Setting the Classpath** for [Windows](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/classpath.html) or [Unix](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/classpath.html).)

# 应用类加载器如何查找 User classes？

为了查找 User classes，Java Launcher 启动程序将引用 *User Classpath* - 一个包含了用户定义的类文件的目录、jar 包和 zip 包列表。作为 Java 虚拟机或 Java 编译器中的一个参数，可以在命令行上或通过环境变量进行设置。

*User Classpath* 使用字符串格式指定，路径不要含有空格，否则转义为 `%20` 之后会报错，例如：

```
java.lang.RuntimeException: Cannot resolve classpath entry: java.lang.RuntimeException: Cannot resolve classpath entry: D:\myprogram\mybatis%20tool\mybatis-generator-gui-0.8.4\target\classes\lib\mysql-connector-java-5.1.38.jar
```

每个路径使用以下方式进行分隔：

* 在类 Unix 系统中，以冒号（`:`）分隔
* 在 Windows 系统中，以分号（`;`）分隔

Java Launcher 启动程序将这个 *User Classpath* 字符串放到 `java.class.path` 系统属性中。该值的来源及优先级如下：

* 默认值“ `.`”，表示当前工作目录下的所有类文件（如果在 jar 包中，则位于其下）。
* `CLASSPATH` 环境变量，覆盖默认值。查看方式：`echo $CLASSPATH`，设置方式：`set CLASSPATH=`
* `-cp` 或 `-classpath` 命令行选项，覆盖默认值以及 `CLASSPATH` 环境变量。
* 由 `-jar` 选项指定的 jar 包，它覆盖上述所有值。如果使用此选项，则所有用户类必须来自指定的 jar 包。

类文件具有反映“类的完全限定名称（class's fully-qualified name）”的子路径名。例如，如果类 `com.mypackage.MyClass` 存储在 `/myclasses` 目录，则 `/myclasses` 必须在 *User Classpath* 中，并且类文件的完整路径必须为 `/myclasses/com/mypackage/MyClass.class`。

如果类存储在名为 `myclasses.jar` 的 jar 包中，则 `myclasses.jar` 必须在  *User Classpath*  中，并且类文件必须存储 `myclasses.jar/com/mypackage/MyClass.class`。

下面来看几个例子，总结如下：

![CLASSPATH 例子](/img/java/classloader/User_Classpath.png)

## Unpacked Classes

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

* 永久设置：使用 `CLASSPATH` 环境变量
* 临时设置：使用 `-cp` 或 `-classpath` 命令行选项

## JAR files

* 单个 jar 包：使用绝对路径指定具体某个 jar 包
* 多个 jar 包：使用绝对路径加上通配符 `*`

## META-INF/MANIFEST.MF

如果程序已经打成 jar 包，需要使用[清单文件](https://en.wikipedia.org/wiki/Manifest_file)指定入口类及 `CLASSPATH`，并使用 `java -jar` 命令启动。例如 Tomcat 的 `bootstrap.jar` 引导包：

```
......
Main-Class: org.apache.catalina.startup.Bootstrap
Class-Path: commons-daemon.jar
```

# IDEA 如何查找类？

首先，为 IDEA 平台配置上你所拥有的 JDK：

![Platform Settings SDKs](/img/java/idea/platform_sdks.png)

然后，为你的项目指定一个默认 SDK：

![Project SDK](/img/java/idea/project_sdk.png)

搞掂之后，IDEA 会为项目载入指定版本的 SDK，将基础目录  `jre/lib/` 的 Bootstrap classes 和扩展目录  `jre/lib/ext/` 的 Extension classes 加入 classpath：

![External Libraries](/img/java/idea/external_libraries.png)

# 参考

JavaDoc

- https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html

- https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html

- https://docs.oracle.com/javase/8/docs/technotes/tools/findingclasses.html

- https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html
- http://cr.openjdk.java.net/~mr/jigsaw/ea/module-summary.html
- [The Class Loader and Class File Verifier](http://medialab.di.unipi.it/web/doc/JNetSec/jns_ch5.htm)

Wikipedia

- https://en.wikipedia.org/wiki/Classpath_(Java)
- https://en.wikipedia.org/wiki/Java_class_file
- https://en.wikipedia.org/wiki/Java_Class_Library

其它：

- https://blog.csdn.net/briblue/article/details/54973413

- https://www.cnblogs.com/doit8791/p/5820037.html

- 《[理解java对象初始化顺序](https://www.cnblogs.com/Kidezyq/p/11769839.html)》先父后子，先静后动

- 《[原来热加载如此简单，手动写一个 Java 热加载吧](https://mp.weixin.qq.com/s/iGKprJEqCZpIO77sAMYRCQ)》

- https://zhuanlan.zhihu.com/p/54693308