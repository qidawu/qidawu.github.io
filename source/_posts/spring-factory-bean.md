---
title: Spring Factory Bean 总结
date: 2017-06-15 22:26:34
updated:
tags: [Java, Spring]
typora-root-url: ..
---

# 简介

[org.springframework.beans.factory.FactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/FactoryBean.html) 用于在 IoC 容器中创建其它 Bean，该接口定义如下：

```java
T getObject()  // Return an instance (possibly shared or independent) of the object managed by this factory.
Class<?> getObjectType()  // Return the type of object that this FactoryBean creates, or null if not known in advance.
boolean isSingleton()  // Is the object managed by this factory a singleton? That is, will getObject() always return the same object (a reference that can be cached)?
```

有哪些现存的 `FactoryBean`？例如：

* 当需要从 JNDI 查找对象（例如 `DataSource`）时，可以使用 `JndiObjectFactoryBean`。
* 当使用 Spring AOP 为 bean 创建代理时，可以使用 `ProxyFactoryBean`。
* 当需要在 IoC 容器中创建 Hibernate 的 `SessionFactory` 时，可以使用 `LocalSessionFactoryBean`。
* 当需要在 IoC 容器中创建 MyBatis 的 `SqlSessionFactory` 时，可以使用 `SqlSessionFactoryBean`。

# 例子

如果要为某个接口生成 JDK 动态代理，且将该代理对象放入 Spring IoC 容器，以便后续依赖注入使用，可以自定义实现 `FactoryBean` 实现如下效果，如图：

![HttpApiService_example](/img/spring/HttpApiService_example.png)

实现代码如下：

```java
/**
 * Xxx 接口
 */
public interface HttpApiService {
    HttpRespDTO<XxxRespDTO> api1(XxxReqDTO reqDTO);	
}

/**
 * Xxx 接口工厂，用于创建代理实现
 */
@Configuration
public class HttpApiServiceFactoryBean implements FactoryBean<HttpApiService> {

    private static final Class<?> API_INTERFACE = HttpApiService.class;

    @Override
    public HttpApiService getObject() {
        return (HttpApiService) Proxy.newProxyInstance(
            API_INTERFACE.getClassLoader(), new Class[]{API_INTERFACE}, new HttpApiServiceProxy());
    }

    @Override
    public Class<?> getObjectType() {
        return API_INTERFACE;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}

/**
 * Xxx 接口的动态代理实现
 */
public class HttpApiServiceProxy implements InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        ......
    }
}
```

# 参考

《[Spring BeanFactory和FactoryBean的区别](https://www.jianshu.com/p/05c909c9beb0)》

[org.springframework.beans.factory.FactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/FactoryBean.html)

[ThreadPoolExecutorFactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolExecutorFactoryBean.html)

[LocalSessionFactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/orm/hibernate5/LocalSessionFactoryBean.html)

[SqlSessionFactoryBean](https://github.com/mybatis/spring/blob/master/src/main/java/org/mybatis/spring/SqlSessionFactoryBean.java)
