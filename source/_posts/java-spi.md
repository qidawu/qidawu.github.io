---
title: Java 类加载篇（二）Java SPI 机制总结
date: 2018-11-05 22:27:20
updated:
tags: Java
typora-root-url: ..
---

# Java SPI

SPI 全称 Service Provider Interface，Java 1.6 引入，是 Java 在语言层面为我们提供了一种方便地创建可扩展应用的途径。SPI 提供了一种 JVM 级别的服务发现机制，我们只需要按照 SPI 的要求，在 jar 包中进行适当的配置，JVM 就会在运行时通过懒加载，帮我们找到所需的服务并加载。如果我们一直不使用某个服务，那么它不会被加载，一定程度上避免了资源的浪费。

整体机制图如下：

![Java SPI](/img/java/spi/java-spi.webp)

## 使用例子

以 JDBC 为例，**标准服务接口**为 `com.mysql.jdbc.Driver`。

MySQL 作为**服务提供方**，以 mysql-connector-java 5.1.44 为例，按规范要求其 `META-INF/services/java.sql.Driver` 配置文件中声明了两个**实现类**，如下：

```
com.mysql.jdbc.Driver
com.mysql.fabric.jdbc.FabricMySQLDriver
```

当类加载器载入 `java.sql.DriverManager` 类时，会执行其静态代码块，从而执行其中的 SPI 代码加载 JDBC Driver 实现，源码如下，详见：[《Java 数据持久化系列（一）JDBC Driver 驱动程序总结》](/2019/01/23/java-jdbc-driver/)

```java
ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
Iterator<Driver> driversIterator = loadedDrivers.iterator();
while(driversIterator.hasNext()) {
    Driver driver = driversIterator.next();
}
```

流程如下：

![spi_flow_diagram](/img/java/spi/spi_flow_diagram.png)

## 源码解析

`ServiceLoader` 的结构如下：

![ServiceLoader](/img/java/spi/ServiceLoader.png)

其成员变量如下：

```java
public final class ServiceLoader<S>
    implements Iterable<S>
{

    // 类加载器加载配置文件时 所用的固定目录
    private static final String PREFIX = "META-INF/services/";

    // 代表被加载的类或者接口
    // The class or interface representing the service being loaded
    private final Class<S> service;

    // 用于定位，加载和实例化 providers 的类加载器
    // The class loader used to locate, load, and instantiate providers
    private final ClassLoader loader;

    // 创建 ServiceLoader 时采用的访问控制上下文
    // The access control context taken when the ServiceLoader is created
    private final AccessControlContext acc;

    // 缓存 providers，按实例化的顺序排列
    // Cached providers, in instantiation order
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // 懒查找迭代器
    // The current lazy-lookup iterator
    private LazyIterator lookupIterator;
}
```

SPI 的核心在于内部类 `LazyIterator`，承担了以下职责：

1. 加载配置文件，解析、验证其内容
2. 加载类
3. 反射构造实例

核心源码及注释如下：

```java
    // Private inner class implementing fully-lazy provider lookup
    //
    private class LazyIterator
        implements Iterator<S>
    {

        Class<S> service;
        ClassLoader loader;
        Enumeration<URL> configs = null;
        Iterator<String> pending = null;
        String nextName = null;

        private boolean hasNextService() {
            if (nextName != null) {
                return true;
            }
            // 判断是否首次使用
            if (configs == null) {
                try {
                    // 本例中值为 META-INF/services/java.sql.Driver
                    String fullName = PREFIX + service.getName();
                    if (loader == null)
                        configs = ClassLoader.getSystemResources(fullName);
                    else
                        // 使用类加载器从类路径中加载文件：META-INF/services/java.sql.Driver，如果多个 jar 包都存在该文件则结果为多个 URL 实例
                        configs = loader.getResources(fullName);
                } catch (IOException x) {
                    fail(service, "Error locating configuration files", x);
                }
            }
            while ((pending == null) || !pending.hasNext()) {
                if (!configs.hasMoreElements()) {
                    return false;
                }
                // 依次解析 URL，获取 URL 内容的迭代器
                pending = parse(service, configs.nextElement());
            }
            // 依次获取 URL 内容，例如第一条为 com.mysql.jdbc.Driver
            nextName = pending.next();
            return true;
        }

        private S nextService() {
            if (!hasNextService())
                throw new NoSuchElementException();
            String cn = nextName;
            nextName = null;
            Class<?> c = null;
            try {
                // 使用指定的类加载器查找并加载类：com.mysql.jdbc.Driver
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
                fail(service,
                     "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
                fail(service,
                     "Provider " + cn  + " not a subtype");
            }
            try {
                // 通过反射，调用 com.mysql.jdbc.Driver 的 public 无参构造方法创建 Object 实例对象，并强制转换为 interface java.sql.Driver 类型
                S p = service.cast(c.newInstance());
                // 塞入缓存，key 为 com.mysql.jdbc.Driver 字符串，value 是对应的实例对象
                providers.put(cn, p);
                return p;
            } catch (Throwable x) {
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }

        ...

}
```

# 使用场景

Java SPI 实际上是“**基于接口的编程＋策略模式＋配置文件**”组合实现的动态加载机制。这种动态加载机制的使用场景如下：

* JDBC Driver 驱动程序管理类 [`java.sql.DriverManager`](https://docs.oracle.com/javase/8/docs/api/java/sql/DriverManager.html)。详见：[JDBC Driver 驱动程序总结](/2018/02/03/java-jdbc-driver/)
* JSR-303 Bean Validation 的 [`javax.validation.Validation`](https://docs.oracle.com/javaee/7/api/javax/validation/Validation.html)
* 日志门面接口实现类加载，SLF4J 加载不同提供商的日志实现类。
* Spring
  * 对 servlet3.0 规范对 `ServletContainerInitializer` 的实现
  * 自动类型转换 Type Conversion SPI (Converter SPI、Formatter SPI) 等
  * Spring Factories 机制（`SpringFactoriesLoader`）
* Dubbo 通过 SPI 机制加载所有的组件。不过，Dubbo 并未使用 Java 原生的 SPI 机制，而是对其进行了增强，使其能够更好的满足需求。在 Dubbo 中，SPI 是一个非常重要的模块。基于 SPI，我们可以很容易的对 Dubbo 进行拓展。如果大家想要学习 Dubbo 的源码，SPI 机制务必弄懂。详见：http://dubbo.apache.org/zh-cn/docs/source_code_guide/dubbo-spi.html

# 对比总结

下面总结下这几个加载类：

* Java `java.util.ServiceLoader`
* Spring `org.springframework.core.io.support.SpringFactoriesLoader`
* Dubbo `com.alibaba.dubbo.common.extension.ExtensionLoader`

|          | Java SPI                         | Spring Factories                                    | Dubbo SPI                                               |
| -------- | -------------------------------- | --------------------------------------------------- | ------------------------------------------------------- |
| 加载类   | `ServiceLoader`                  | `SpringFactoriesLoader`                             | `ExtensionLoader`                                       |
| 加载文件 | `META-INF/services/接口全限定名` | `META-INF/spring.factories`                         | `META-INF/dubbo`                                        |
| 文件内容 | 接口实现类，多值以**换行**分隔   | 通过键值对方式（key=value）配置，多值以**逗号**分隔 | 通过键值对方式（key=value）配置，支持按需加载接口实现类 |
| 接口注解 | /                                | /                                                   | `@SPI`                                                  |

# 参考

https://www.jianshu.com/p/46b42f7f593c