---
title: Java 反射篇（四）动态代理之 JDK Proxy 总结
date: 2018-11-31 22:53:54
updated:
tags: Java
typora-root-url: ..
---

什么是代理？

> 代理对象 = 增强代码 + 目标对象

![proxy](/img/java/proxy/Proxy.png)

有哪些代理方式？

* 静态代理
* 动态代理

什么是动态代理？

> 动态代理是一种在**运行时**动态生成代理的机制。这个概念是与静态代理相对的，静态代理需要为每一个目标类都手工编写或用工具生成一个对应的代理类，非常繁琐。

动态代理的实现方式？

> 动态代理的实现方式有很多种，比如：
>
> * 利用 JDK 自身提供的动态代理 API（`java.lang.reflect`）
>
> * 或者利用性能更高的第三方字节码生成框架（例如 ASM、cglib（基于 ASM）、Javassist 等）
>
> 最终目标都是生成一个代理类的字节码。

哪些场景用到动态代理？

> 比如：
>
> 包装 RPC 调用、…
>
> 面向切面的编程（AOP）
>
> ![](/img/spring/aop/aop_lib.png)

动态代理从代理对象创建到方法执行的整体流程如下：

![jdk_proxy_process](/img/java/proxy/jdk_proxy_process.png)

下面来看下 JDK 自身提供的动态代理，底层是如何实现的。

# 例子

来个例子，实现如图效果：

![jdk_proxy_process2](/img/java/proxy/jdk_proxy_example.png)

首先创建接口 `Flyable` 和目标类 `Bird`：

```java
/**
 * 目标类和代理类共同实现的接口
 **/
public interface Flyable {
    void fly(String param);
}

/**
 * 目标类
 **/
@Slf4j
public class Bird implements Flyable {
    @Override
    public void fly(String param) {
        log.info("Target bird fly, param = {}", param);
    }
}
```

然后是关键的一步，实现 `InvocationHandler` 接口，创建代理类：

```java
/**
 * 代理类
 **/
@Slf4j
@AllArgsConstructor
public class BirdProxy implements InvocationHandler {

    // 目标对象
    private Flyable target;

    // proxy 参数表示动态生成的 Proxy 类 通过反射创建出来的对象
    // method 参数表示 proxy 对象本次执行的方法，可以判断该参数动态决定执行对应的业务逻辑
    // args 参数表示 proxy 对方本次执行的方法参数
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 在目标对象执行前执行代码
        log.info("Before target bird method: {}", method.getName());
        // 在目标对象上执行指定方法
        Object result = method.invoke(target, args);
        // 在目标对象执行后执行代码
        log.info("After target bird method: {}", method.getName());
        return result;
    }

    // 创建代理对象
    public static Flyable newProxy(Flyable target) {
        // 方式一：显式使用反射创建代理对象（先获取 com.sun.proxy.$Proxy0 的 Class 对象）
        // Class<?> proxyClass = Proxy.getProxyClass(Flyable.class.getClassLoader(), target.getClass().getInterfaces());
        // Constructor<?> constructor = proxyClass.getConstructor(InvocationHandler.class);
        // return (Flyable) constructor.newInstance(new BirdProxy(target));
        
        // 方式二：隐式使用反射创建代理对象，API 更简单
        return (Flyable) Proxy.newProxyInstance(Flyable.class.getClassLoader(), target.getClass().getInterfaces(), new BirdProxy(target));
    }

}
```

测试代码：

```java
@Slf4j
public class FlyTest {

    @Test
    public void test() {
        // 创建目标对象
        Bird target = new Bird();
        // 创建代理对象
        Flyable proxy = BirdProxy.newProxy(target);
        // 调用任意方法，将执行代理逻辑
        proxy.fly("hello world");
    }
}
```

输出结果如下：

```java
Before target bird method: fly
Target bird fly, param = hello world
After target bird method: fly
```

# 源码解析

例子中涉及到两个 API，由 Java 1.3 引入：

`java.lang.reflect.InvocationHandler`，代理对象内部的成员变量。作为代理对象和目标对象的**桥梁**，代理对象的每个方法调用，都会调用其 `invoke()` 方法，委托其去调用目标对象，可在此时机补充**增强代码**。

![InvocationHandler](/img/java/proxy/InvocationHandler_class.png)

`java.lang.reflect.Proxy`，用于创建代理类或代理对象，同时还是它们的父类。

![Proxy](/img/java/proxy/Proxy_class.png)

`Proxy` 的核心方法 `newProxyInstance` 用于运行时动态生成代理类并通过反射创建实例，其源码及关键注释如下：

```java
public class Proxy implements java.io.Serializable {

    /** 代理类构造方法的参数类型 */
    private static final Class<?>[] constructorParams = { InvocationHandler.class };

    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * 查找或生成指定的代理类
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * 反射调用代理类的构造方法（入参为指定的 invocation handler）创建实例
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

            // 反射获取构造方法
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            // 调用构造方法，创建代理对象
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }

    /**
     * Generate a proxy class.  Must call the checkProxyAccess method
     * to perform permission checks before calling this.
     */
    private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

        // If the proxy class defined by the given loader implementing
        // the given interfaces exists, this will simply return the cached copy;
        // otherwise, it will create the proxy class via the ProxyClassFactory
        return proxyClassCache.get(loader, interfaces);
    }
}
```

# 获取动态代理生成的 Class 文件

`java.lang.reflect.Proxy` 底层使用了 `sun.misc.ProxyGenerator` 工具类生成代理类。通过指定 `java` 命令参数 `-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true` 可以让工具类将动态生成的字节码写到本地磁盘文件（$ProxyN.class）。本例生成的字节码文件反编译后源码如下，重点关注 `fly` 方法：

```java
package com.sun.proxy;

import com.github.proxy.Flyable;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements Flyable {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void fly(String var1) throws  {
        try {
            // 调用 proxy 对象的 fly 方法，则委托 InvocationHandler 对象执行 invoke 方法
            super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("com.github.proxy.Flyable").getMethod("fly", Class.forName("java.lang.String"));
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

可见，动态生成的 `$Proxy0` 类同样实现了 `Flyable` 接口，与目标类 `Bird` 类形成一个三角结构：

![jdk_proxy](/img/java/proxy/jdk_proxy.png)

`fly` 方法的实现，仅仅只是调用了 `InvocationHandler` 对象的 `invoke` 方法，传入上下文参数。具体的业务逻辑还是在自己的 `InvocationHandler` 中根据参数判断并自行实现。

# 参考

《[Proxy pattern 设计模式](https://en.wikipedia.org/wiki/Proxy_pattern)》

《[代理设计模式](https://refactoringguru.cn/design-patterns/proxy)》

《[10分钟看懂动态代理设计模式](https://www.jianshu.com/p/fc285d669bc5)》

《[动态代理是基于什么原理？](https://time.geekbang.org/column/article/7489)》

《[Java 动态代理作用是什么？ - 知乎用户的回答](https://www.zhihu.com/question/20794107/answer/658139129)》

《[Understanding “proxy” arguments of the invoke method of java.lang.reflect.InvocationHandler](http://stackoverflow.com/questions/22930195/understanding-proxy-arguments-of-the-invoke-method-of-java-lang-reflect-invoca)》

《[字节码增强技术探索](https://tech.meituan.com/2019/09/05/java-bytecode-enhancement.html)》

