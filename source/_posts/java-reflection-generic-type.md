---
title: Java 反射篇（三）泛型 API 总结
date: 2018-11-21 22:53:54
updated:
tags: Java
typora-root-url: ..
---

# 泛型术语

泛型涉及的术语比较多，其与反射接口的对应关系如下：

| 术语                    | 中文含义                 | 举例                               | 反射接口                                   | 备注                                                         |
| ----------------------- | ------------------------ | ---------------------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| Generic type            | 泛型                     | `List<E>`                          | `ParameterizedType`                        |                                                              |
| Parameterized type      | 参数化类型               | `List<String>`                     | `ParameterizedType`                        |                                                              |
| Raw type                | 原始类型                 | `List`                             | `ParameterizedType#getRawType`             | 该方法虽然返回  `Type` 类型，但实际类型是 `Class`，可以强转使用：`(Class<?>) type`。 |
| Unbounded wildcard type | 无限制通配符类型         | `List<?>`                          | `ParameterizedType`                        |                                                              |
| Bounded wildcard type   | 有限制通配符类型（上限） | `List<? extends Number>`           | `ParameterizedType`                        |                                                              |
| Bounded wildcard type   | 有限制通配符类型（下限） | `List<? super Number>`             | `ParameterizedType`                        |                                                              |
| wildcard type           | 通配符类型               | `?`                                | `WildcardType`                             |                                                              |
| Formal type parameter   | 形式类型参数             | `E`                                | `TypeVariable`                             |                                                              |
| Actual type parameter   | 实际类型参数             | `String`                           | `ParameterizedType#getActualTypeArguments` | 该方法虽然返回  `Type[]` 类型，但各元素实际类型是 `Class`，可以强转使用：`(Class<?>) type`。 |
| Bounded type parameter  | 有限制类型参数           | `<E extends Number>`               |                                            |                                                              |
| Recursive type bound    | 递归类型限制             | `<T extends Comparable<T>>`        |                                            |                                                              |
| Generic method          | 泛型方法                 | `static <E> List<E> asList(E[] a)` |                                            |                                                              |
| Type token              | 类型令牌                 | `String.class`                     |                                            |                                                              |

# 泛型 API

## java.lang.reflect.Type

JDK 1.5 引入了泛型特性，一同引入的还有 Java `Type` 类型体系。其中 `java.lang.reflect.Type` 接口作为核心，是 Java 编程语言中所有类型的通用超级接口（common superinterface），这些类型包括：

* 原始类型（raw types）
* 参数化类型（parameterized types）
* 数组类型（array types）
* 八大原始类型（primitive types）
* 类型变量（type variables）

调整后新引入的五个接口如下：

```
java.lang.reflect.Type
  java.lang.reflect.ParameterizedType // 最最常用
  java.lang.reflect.TypeVariable
  java.lang.reflect.WildcardType
  java.lang.reflect.GenericArrayType
```

![Type](/img/java/reflection/Type.png)

它们的核心方法如下：

![Type_methods](/img/java/reflection/Type_methods.png)

类、字段、方法、构造方法也相应增加了一组方法，用于获取 `Type`：

* `java.lang.Class`

  ```java
  // 获取普通 Class
  Class<? super T> getSuperclass()
  Class<?>[] getInterfaces()
  
  // 获取 Type
  Type getGenericSuperclass()
  Type[] getGenericInterfaces()
  ```

* `java.lang.reflect.Field`

  ```java
  // 获取普通 Class
  Class<?> getType()
  
  // 获取 Type
  Type getGenericType()
  ```

* `java.lang.reflect.Method`

  ```java
  // 获取普通 Class
  Class<?> getReturnType()
  Class<?>[] getParameterTypes()
  Class<?>[] getExceptionTypes()
  
  // 获取 Type
  Type getGenericReturnType()
  Type[] getGenericParameterTypes()
  Type[] getGenericExceptionTypes()
  ```

* `java.lang.reflect.Constructor`

  ```java
  // 获取普通 Class
  Class<?>[] getParameterTypes()
  Class<?>[] getExceptionTypes()
  
  // 获取 Type
  Type[] getGenericParameterTypes()
  Type[] getGenericExceptionTypes()
  ```

## java.lang.reflect.GenericDeclaration

同时新增的接口还有 `java.lang.reflect.GenericDeclaration`，用于获取 `TypeVariable`：

![GenericDeclaration_method](/img/java/reflection/GenericDeclaration_method.png)

从源码看，只有三个类实现了该接口（见下图）：

* `java.lang.Class`
* `java.lang.reflect.Method`
* `java.lang.reflect.Constructor`

![GenericDeclaration](/img/java/reflection/GenericDeclaration.png)

因此只有这三个地方可以声明为泛型，并获取其类型参数（type variables）集合：`K`、`V`

* 类型（例如 `class`、`interface`）

  ```java
  public class Test<K, V> { ... }
  
  public interface Test<K, V> { ... }
  ```

* 构造方法（`Constructor`）

  ```java
  public <K, V> Test(K k, V v) { ... }
  ```

* 方法（`Method`）

  ```java
  public <K, V> K test(V v) { ... }
  ```

类、方法、构造方法也相应增加了一个方法，用于获取 `TypeVariable`：

* `java.lang.Class`
* `java.lang.reflect.Method`
* `java.lang.reflect.Constructor`

```java
TypeVariable<?>[] getTypeParameters()
```

# 例子

下面是几个获取 `Type` 的例子，先创建三个类：`BaseMapper`、`PersonMapper`、`Person`

```java
public interface BaseMapper<T extends Serializable & Comparable<T>, K extends Serializable> {

    T getById(K id);

}

public class PersonMapper implements BaseMapper<Person, Long>, Serializable {
    @Override
    public Person getById(Long id) {
        return null;
    }

    public void log(List<?> list) {

    }

    public <T extends Number> void test(List<T> list1, List<? extends Comparable<T>> list2, T[] array, T item) {
    }
}

public class Person implements Serializable, Comparable<Person> {
    private String personName;

    @Override
    public int compareTo(Person o) {
        return this.personName.compareTo(o.getPersonName());
    }
```

## 例子一

```java
    @Test
    public void test1() {
        Method method = PersonMapper.class.getMethod("test", List.class, List.class, Number[].class, Number.class);

        // [0] ParameterizedType
        // [1] ParameterizedType
        // [2] GenericArrayType
        // [3] TypeVariable
        Type[] genericParameterTypes = method.getGenericParameterTypes();
        
        // Class
        Type genericReturnType = method.getGenericReturnType();
        
        // Class[0]
        Type[] genericExceptionTypes = method.getGenericExceptionTypes();
    }
```

三个 `Type` 变量的内容如下：

![generic_type_examples](/img/java/reflection/generic_type_example.png)

## 例子二

接下来看下 `ParameterizedType` 的使用，可以用于获取泛型的原始类型（Raw type）、实际类型参数（Actual type parameter）列表：

```java
    @Test
    public void test2() {
        // com.github.reflection.BaseMapper<com.github.reflection.Person, java.lang.Long>
        Type genericInterface = PersonMapper.class.getGenericInterfaces()[0];
        assertTrue(genericInterface instanceof ParameterizedType);
        ParameterizedType parameterizedType = (ParameterizedType) genericInterface;
        
        // 获取原始类型：BaseMapper.class
        assertEquals(BaseMapper.class, parameterizedType.getRawType());

        // 获取实际类型参数列表：Person.class、Long.class
        Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
        assertEquals("There are two actual type arguments", 2, actualTypeArguments.length);
        // class com.github.reflection.Person
        assertClass("One is Person", actualTypeArguments[0], Person.class);
        // class java.lang.Long
        assertClass("Another is Long", actualTypeArguments[1], Long.class);
    }

    private void assertClass(Type type, Class expectedClass) {
        assertTrue(type instanceof Class);
        Class clazz = (Class) type;
        assertEquals(expectedClass, clazz);
    }
```

`genericInterface` 变量的内容如下，接口返回类型虽然为 `Type`，实际类型为 `ParameterizedType`，因此可以强转。该泛型变量的原始类型、实际类型参数列表如下，实际都为 `Class` 类型，因此可以强转 `(Class<?>) type`：

![ParameterizedType_example](/img/java/reflection/ParameterizedType_example.png)

## 例子三

`WildcardType` 的使用：

```java
    @Test
    public void test3() {
        // java.util.List<?>
        Method method = PersonMapper.class.getMethod("log", List.class);
        Type genericParameterType = method.getGenericParameterTypes()[0];
        assertTrue("The first parameter type of log method is instance of ParameterizedType", 
                   genericParameterType instanceof ParameterizedType);
        ParameterizedType parameterType = (ParameterizedType) genericParameterType;

        // WildcardType
        Type type = parameterType.getActualTypeArguments()[0];
        assertTrue("The actual type argument of ParameterizedType is instance of WildcardType", 
                   type instanceof WildcardType);
        WildcardType wildcardType = (WildcardType) type;
        assertEquals("No lower bounds exist", 0, wildcardType.getLowerBounds().length);
        assertEquals("Only one upper bound exist", 1, wildcardType.getUpperBounds().length);
        // 通配符默认上限类型为 Object
        assertEquals("The upper bound is Object", Object.class, wildcardType.getUpperBounds()[0]);
    }
```

`Type` 类型变量 `genericParameterType` 的内容如下，实际类型是 `ParameterizedType`，其 `?` 通配符是 `WildcardType`：

![WildcardType_example](/img/java/reflection/WildcardType_example.png)

## 例子四

`TypeVariable` 的使用：

```java
    @Test
    public void test4() {
        Method method = BaseMapper.class.getMethod("getById", Serializable.class);

        // TypeVariable
        Type genericReturnType = method.getGenericReturnType();
        assertTrue("Return type of method is instance of TypeVariable", genericReturnType instanceof TypeVariable);
        TypeVariable typeVariable = (TypeVariable) genericReturnType;
        assertEquals("First upper bound is Serializable", Serializable.class, typeVariable.getBounds()[0]);
        ParameterizedType parameterizedType = (ParameterizedType) typeVariable.getBounds()[1];
        assertEquals("Second upper bound is Comparable", Comparable.class, parameterizedType.getRawType());
    }
```

`Type` 类型变量 `genericReturnType` 的内容如下，实际类型是 `TypeVariable`，

![TypeVariable_example](/img/java/reflection/TypeVariable_example.png)

## 例子五

定义一个泛型工具类，用于获取 `T.class`：

```java
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
 
public class GenericsUtils {
    /**
     * 通过反射获得定义 Class 时声明的父类的泛型参数的类型
     * @param clazz 
     * @return 返回第一个类型
     */
    public static Class getSuperClassGenricType(Class clazz) {
        return getSuperClassGenricType(clazz, 0);
    }
 
    /**
     * 通过反射获得定义 Class 时声明的父类的泛型参数的类型
     * @param clazz
     * @param 返回某个下标的类型
     */
    public static Class getSuperClassGenricType(Class clazz, int index)
            throws IndexOutOfBoundsException {
        Type genType = clazz.getGenericSuperclass();
        if (!(genType instanceof ParameterizedType)) {
            return Object.class;
        }
        Type[] params = ((ParameterizedType) genType).getActualTypeArguments();
        if (index >= params.length || index < 0) {
            return Object.class;
        }
        if (!(params[index] instanceof Class)) {
            return Object.class;
        }
        return (Class) params[index];
    }
}
```

# 参考

API：

- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Type.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/ParameterizedType.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/TypeVariable.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/WildcardType.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/GenericArrayType.html

https://www.baeldung.com/java-generics-vs-extends-object

http://tutorials.jenkov.com/java-generics/wildcards.html

https://www.jianshu.com/p/faed45dbfa0c

https://blog.csdn.net/ShuSheng0007/article/details/89520530
