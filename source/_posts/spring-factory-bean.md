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

# 使用方式

这里我们举一个例子：

如果要为某个接口生成 JDK 动态代理，且将该代理对象放入 Spring IoC 容器，以便后续依赖注入使用，可以自定义实现 `FactoryBean` 实现如下效果，如图：

![HttpApiService_example](/img/spring/HttpApiService_example.png)

实现代码如下，首先创建接口及其代理类：

```java
/**
 * Xxx 接口
 */
public interface HttpApiService {
    HttpRespDTO<XxxRespDTO> api1(XxxReqDTO reqDTO);	
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

## 一、手工注册 1.0

如果只需创建一个 `FactoryBean`，可以将其作为一个 Java Config 加上 `@Configuration` 即可：

```java
/**
 * Xxx 接口工厂，用于创建代理实现
 */
@Configuration
public class HttpApiServiceFactoryBean implements FactoryBean<HttpApiService> {

    private static final Class<?> API_INTERFACE = HttpApiService.class;

    @Override
    public HttpApiService getObject() {
        return (HttpApiService) Proxy.newProxyInstance(
            API_INTERFACE.getClassLoader(), new Class[]{ API_INTERFACE }, new HttpApiServiceProxy());
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
```

## 二、手工注册 2.0

如果需要创建多个 `FactoryBean`，可以使用泛型：

```java
@Setter
public class HttpApiServiceFactoryBean<T> implements FactoryBean<T> {

    private Class<T> apiService;

    @SuppressWarnings("unchecked")
    @Override
    public T getObject() {
        return (T) Proxy.newProxyInstance(
                apiService.getClassLoader(), new Class[]{ apiService }, new HttpApiServiceProxy());
    }

    @Override
    public Class<?> getObjectType() {
        return apiService;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

}
```

Java Config 如下，手工创建多个 `FactoryBean`：

```java
@Configuration
public class HttpApiServiceConfig {
    @Bean
    public HttpApiServiceFactoryBean<UserHttpApiService> userHttpApiService() {
        return new HttpApiServiceFactoryBean(UserHttpApiService.class);
    }
    
    @Bean
    public HttpApiServiceFactoryBean<RoleHttpApiService> roleHttpApiService() {
        return new HttpApiServiceFactoryBean(RoleHttpApiService.class);
    }
}
```

这种方式参考了 [Mybatis-Spring 注册映射器](https://mybatis.org/spring/zh/mappers.html#register)。

## 三、自动发现

如果想进一步省略 Java Config，做到自动扫描并创建 `FactoryBean`，可以创建自动配置类。例如，为添加了 `@HttpApi` 注解的接口创建相应的 `FactoryBean`：

```java
/**
 * HttpApi 动态代理自动配置类
 **/
@Configuration
@Import({AutoConfiguredHttpApiScannerRegistrar.class})
public class HttpApiServiceAutoConfig {
}

/**
 * 自定义 ImportBeanDefinitionRegistrar
 **/
@Slf4j
class AutoConfiguredHttpApiScannerRegistrar implements ImportBeanDefinitionRegistrar, ResourceLoaderAware {

    // 扫描 Application 类所在的 classpath。也可以指定其它路径（如配合 @EnableHttpApi 注解使用）
    private static final String BASE_PKG = Application.class.getPackage().getName();
    private ResourceLoader resourceLoader;

        @Override
        public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
            try {
                log.debug("Searching for HttpApiService annotated with @HttpApi from {}", BASE_PKG);
                
                // 配置自定义的 ClassPathHttpApiScanner
                ClassPathHttpApiScanner scanner = new ClassPathHttpApiScanner(registry);
                if (this.resourceLoader != null) {
                    scanner.setResourceLoader(this.resourceLoader);
                }
                // 扫描指定路径
                scanner.doScan(BASE_PKG);
            } catch (IllegalStateException ex) {
                log.debug("Could not determine auto-configuration package, automatic @HttpApi scanning disabled.", ex);
            }
        }

        @Override
        public void setResourceLoader(ResourceLoader resourceLoader) {
            this.resourceLoader = resourceLoader;
        }
    }

/**
 * 自定义 ClassPathHttpApiScanner
 **/
class ClassPathHttpApiScanner extends ClassPathBeanDefinitionScanner {

    public ClassPathHttpApiScanner(BeanDefinitionRegistry registry) {
        super(registry, false);
    }

    @Override
    protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
        // 添加过滤条件，这里是只要添加了 @HttpApi 注解的类或接口，就会被扫描到
        addIncludeFilter(new AnnotationTypeFilter(HttpApi.class));

        // 调用 Spring 的扫描
        Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);

        if (beanDefinitions.isEmpty()) {
            logger.warn( "No @HttpApi bean was found in '" + Arrays.toString(basePackages) + "' package. Please check your configuration.");
        }
        // 处理扫到的 BeanDefinition
        else {
            processBeanDefinitions(beanDefinitions);
        }
        return beanDefinitions;
    }

    private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {
        GenericBeanDefinition definition;
        for (BeanDefinitionHolder holder : beanDefinitions) {
            definition = (GenericBeanDefinition) holder.getBeanDefinition();

            String beanClassName = definition.getBeanClassName();
            
            // 修改 BeanDefinition 的 BeanClass 为 FactoryBean
            definition.setBeanClass(HttpApiServiceFactoryBean.class);

            // 为构造方法指定所需参数
            // definition.getPropertyValues().add("restTemplate", new RuntimeBeanReference("restTemplate"));
            definition.getPropertyValues().add("apiService", getClass(beanClassName));
        }
    }

    private Class<?> getClass(String beanClassName){
        try {
            return Class.forName(beanClassName);
        } catch (ClassNotFoundException e) {
            throw new IllegalStateException(e);
        }
    }

    /**
     * 覆盖原有策略，限定只需要接口类型
     */
    @Override
    protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
        return beanDefinition.getMetadata().isInterface() && beanDefinition.getMetadata().isIndependent();
    }

}
```

这种方式参考了 [Mybatis-Spring 发现映射器](https://mybatis.org/spring/zh/mappers.html#scan)。

需要用到 Spring 的几个类，待补充：

* `ImportBeanDefinitionRegistrar`
* `ClassPathBeanDefinitionScanner`
  * `BeanDefinitionHolder`
  * `GenericBeanDefinition`
  * `AnnotatedBeanDefinition`

![BeanDefinition](/img/spring/BeanDefinition.png)

# 参考

《[Spring BeanFactory和FactoryBean的区别](https://www.jianshu.com/p/05c909c9beb0)》

[org.springframework.beans.factory.FactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/FactoryBean.html)

[ThreadPoolExecutorFactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolExecutorFactoryBean.html)

[LocalSessionFactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/orm/hibernate5/LocalSessionFactoryBean.html)

[SqlSessionFactoryBean](https://github.com/mybatis/spring/blob/master/src/main/java/org/mybatis/spring/SqlSessionFactoryBean.java)

[MyBatis-Spring 注入映射器](https://mybatis.org/spring/zh/mappers.html#)