---
title: Java 类加载篇（一）ClassLoader 类加载机制总结
date: 2018-11-01 21:04:43
updated:
tags: Java
typora-root-url: ..
---

# 类的加载过程

.java 源文件的从编译、加载、到对象创建的过程如下：

![classloaoder_process](/img/java/classloader/classloaoder_process.jpg)

* 首先，.java 源文件经过 `javac` 编译后生成 .class 类文件（内含字节码）。
* 然后，通过 `jar` 命令或其它构建工具（如 Maven、Gradle）打包生成可运行的 jar 包。
* 最终，通过 `java -jar` 命令运行 jar 包，执行其中清单文件（`META-INF/MANIFEST.MF`）中通过 `Main-Class` 指定的入口类的 `main` 方法以启动程序，并按照其 `Class-Path` 设置类路径。

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

### loadClass (双亲委派)

`loadClass` 方法使用二进制名称（[binary name](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#name)）、通过“**双亲委派模型（Delegation Model）**”自顶向下尝试加载类，如下图所示：

![classloader_hierarchy](/img/java/classloader/classloader_hierarchy.png)

⭐️ 这种设计的好处体现在：

* 沙箱安全机制：例如自己写的 `java.lang.String` 类不会被加载，否则在 `defineClass` 方法这一步会报错，**防止恶意代码污染，核心 API 库被随意篡改。**核心 API 库只能由 `Bootstrap ClassLoader` 从`$JAVA_HOME/jre/lib` 目录进行加载。
* 避免类的重复加载：当父加载器已经加载了该类时，就没有必要再加载一次，**保证被加载类的唯一性。**

类加载过程如下：

1. 调用自身的 [`findLoadedClass(String)`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#findLoadedClass-java.lang.String-) 方法以检查类是否已经被加载。
2. 如未，则递归调用父加载器的 [`loadClass`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#loadClass-java.lang.String-) 方法（如果父加载器为 `null`，则使用虚拟机内置的 Bootstrap ClassLoader）。 
3. 如果父加载器都加载不到，则调用自身的 [`findClass(String)`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#findClass-java.lang.String-) 方法查找类。
4. 如果上述步骤找到了类，并且 `resolve` 标记为 `true`，则在目标 `Class` 对象上调用 [`resolveClass(Class)`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html#resolveClass-java.lang.Class-) 方法。

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

![ClassLoader](/img/java/classloader/ClassLoader.png)

可以通过继承 `ClassLoader` 类，实现一个自定义的类加载器（参考 [`NetworkClassLoader`](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html) 例子）。通过重写 `findClass` 方法可以从以下途径查找类文件：

- 从 ZIP 包中读取，最常见，JAR，WAR，EAR 格式的基础。
- 从网络中获取，典型场景是 Applet。
- 运行时计算生成，典型情景是 JDK 动态代理技术。
- 从其它文件中生成，典型场景是 JSP 应用，即由 JSP 文件生成对应的 Servlet Class 类。

也可以使用自带的 `URLClassLoader` 从本地路径（`file:/`）或网络路径（`http://`）加载类文件，示例如下：

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

类可以按需**动态加载**到内存，这是 Java 的一大特点，也称为运行时绑定，或动态绑定。

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

进入类加载的“连接（Linking）”阶段（详见官方文档 ["Execution" chapter of The Java™ Language Specification](https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html)）。

## 资源加载

使用场景：例如 Spring Factories 机制中 `SpringFactoriesLoader` 加载类路径下的文件：

```java
classLoader.getResources("META-INF/spring.factories")
```

文件加载后，通过 key-value 的方式读取指定 key，并以反射的方式实例化指定的类型。

### getResource

查找指定名称的资源（图像、音频、文本等）。资源的名称是用“/”分隔的路径名，用于标识资源。
该方法首先递归调用父加载器查找资源；如果父加载器为 `null`，则使用虚拟机内置的启动类加载器（Bootstrap ClassLoader）。如果父加载器查找失败，则调用自身的 `findResource(String)` 查找资源。整个资源加载过程仍然为“双亲委派模型（Delegation Model）”：

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

# 参考

JavaDoc

- [How Classes are Found](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/findingclasses.html)
- https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html
- [Java Language Specification - Chapter 12. Execution](https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html)
- https://docs.oracle.com/javase/8/docs/technotes/tools/findingclasses.html
- https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html

- [The Class Loader and Class File Verifier](http://medialab.di.unipi.it/web/doc/JNetSec/jns_ch5.htm)

其它：

- [一看你就懂，超详细 java 中的 ClassLoader 详解](https://blog.csdn.net/briblue/article/details/54973413)

- [Java 类加载器 ClassLoader 总结](https://www.cnblogs.com/doit8791/p/5820037.html)

- 《[理解java对象初始化顺序](https://www.cnblogs.com/Kidezyq/p/11769839.html)》先父后子，先静后动

- 《[原来热加载如此简单，手动写一个 Java 热加载吧](https://mp.weixin.qq.com/s/iGKprJEqCZpIO77sAMYRCQ)》

- https://zhuanlan.zhihu.com/p/54693308

