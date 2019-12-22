---
title: Java 反射篇（一）基础 API 总结
date: 2018-11-07 22:53:54
updated:
tags: Java
typora-root-url: ..
---

什么是反射？

> 反射机制是 Java 语言提供的一种基础功能，赋予程序在**运行时自省**（introspect，官方用语）的能力。通过反射我们可以直接操作类或者对象，比如获取某个对象的类定义，获取类声明的属性和方法，调用方法或者构造对象，甚至可以运行时修改类定义。

主要类：

```java
java.lang.Class
java.lang.Package
java.lang.reflect.Field
java.lang.reflect.Method
java.lang.reflect.Constructor
// JDK 1.8 新引入，通过该类的 getName 方法能在运行时得到参数的名称（前提是通过 -parameters 指定编译器在编译的时候将参数名编译进去）
java.lang.reflect.Parameter

java.lang.reflect.Array
```

工具类：

```
java.lang.reflect.Modifier
```

继承关系：

![main_api](/img/java/reflection/main_api.png)

# Class 类

如何获取一个 `class` 的 `Class` 实例？有几种方法：

```java
// 方法一：直接通过一个 class 的静态变量 class 获取
Class<String> aClass = String.class;

// 方法二：通过实例变量提供的 getClass() 方法获取
Class<? extends String> aClass2 = "Hello".getClass();

// 方法三：通过 class 的完整类名获取，底层调用的是应用类加载器 Application ClassLoader (sun.misc.Launcher$AppClassLoader)
Class<?> aClass3 = Class.forName("java.lang.String");

// 方法四：通过自定义 ClassLoader 获取
URLClassLoader classLoader = new URLClassLoader(new URL[] {new URL("file:/c:/")});
Class<?> aClass4 = classLoader.loadClass("java.lang.String");

// 获取父类
Class<? super String> superclass = String.class.getSuperclass();  // class java.lang.Object
// 获取外部类
Class<?> enclosingClass = Map.Entry.class.getEnclosingClass();  // interface java.util.Map
// 获取内部类
Class<?>[] classes = HashMap.class.getClasses();  // 2 个内部类
Class<?>[] declaredClasses = HashMap.class.getDeclaredClasses();  // 13 个内部类
```

由于 `Class` 实例在 JVM 中是唯一的，因此，上述方法获取的 `Class` 实例都是同一个实例。可以用 `==` 比较两个 `Class` 实例求证：

```java
// true
assertTrue(aClass == aClass2);
assertTrue(aClass == aClass3);
assertTrue(aClass2 == aClass3);
```

拿到 `Class` 实例后，可以通过下列方法获取字段、方法、构造方法、注解，以进行后续操作。方法名以 `getDeclared` 开头的表示仅获取当前类的信息（不包括父类）：

| Member        | Class API                        | Param  type             | Return type | Inherited members | Private members |
| ------------- | -------------------------------- | ----------------------- | ----------- | ----------------- | --------------- |
| `Class`       | `getDeclaredClasses()`           |                         | Array       | N                 | Y               |
|               | `getClasses()`                   |                         | Array       | Y                 | N               |
| `Field`       | `getDeclaredField()`             | `String`                | Single      | N                 | Y               |
|               | `getField()`                     | `String`                | Single      | Y                 | N               |
|               | `getDeclaredFields()`            |                         | Array       | N                 | Y               |
|               | `getFields()`                    |                         | Array       | Y                 | N               |
| `Method`      | `getDeclaredMethod()`            | `String`, `Class<?>...` | Single      | N                 | Y               |
|               | `getMethod()`                    | `String`, `Class<?>...` | Single      | Y                 | N               |
|               | `getDeclaredMethods()`           |                         | Array       | N                 | Y               |
|               | `getMethods()`                   |                         | Array       | Y                 | N               |
| `Constructor` | `getDeclaredConstructor()`       | `Class<?>...`           | Single      | N/A               | Y               |
|               | `getConstructor()`               | `Class<?>...`           | Single      | N/A               | N               |
|               | `getDeclaredConstructors()`      |                         | Array       | N/A               | Y               |
|               | `getConstructors()`              |                         | Array       | N/A               | N               |
| `Annotation`  | `getDeclaredAnnotation()`        | `Class<T>`              | Single      | N                 | N/A             |
|               | `getAnnotation()`                | `Class<T>`              | Single      | Y                 | N/A             |
|               | `getDeclaredAnnotationsByType()` | `Class<T>`              | Array       | N                 | N/A             |
|               | `getAnnotationsByType()`         | `Class<T>`              | Array       | Y                 | N/A             |
|               | `getDeclaredAnnotations()`       |                         | Array       | N                 | N/A             |
|               | `getAnnotations()`               |                         | Array       | Y                 | N/A             |

# 字段

一个 `Field` 对象包含了一个字段的所有信息，例如：

```java
// 字段名称
String getName()
// 字段类型
Class<?> getType()
// 字段的修饰符，返回值是一个int，不同的 bit 表示不同的含义。使用 Modifier 工具类进行解析
int getModifiers()
```

以 `String` 类的 `value` 字段为例，它的定义是：

```java
public final class String {
    private final byte[] value;
}
```

用反射获取该字段的信息，代码如下：

```java
Field f = String.class.getDeclaredField("value");
f.getName(); // "value"
f.getType(); // class [B 表示byte[]类型
int m = f.getModifiers();
Modifier.isFinal(m); // true
Modifier.isPublic(m); // false
Modifier.isProtected(m); // false
Modifier.isPrivate(m); // true
Modifier.isStatic(m); // false
```

获取或设置字段值，使用如下方法。注意如果试图获取非 `public` 字段，将抛出异常 `java.lang.IllegalAccessException`，解决办法是先设置 `setAccessible(true)`：

```java
// 获取字段值
Object get(Object obj)
boolean getBoolean(Object obj)
byte getByte(Object obj)
char getChar(Object obj)
short getShort(Object obj)
int getInt(Object obj)
long getLong(Object obj)
float getFloat(Object obj)
double getDouble(Object obj)

// 设置字段值
void set(Object obj, Object value)
void setBoolean(Object obj, boolean z)
void setByte(Object obj, byte b)
void setChar(Object obj, char c)
void setShort(Object obj, short s)
void setInt(Object obj, int i)
void setLong(Object obj, long l)
void setFloat(Object obj, float f)
void setDouble(Object obj, double d)
```

# 方法

一个 `Method` 对象包含一个方法的所有信息，例如：

```java
// 方法名称
String getName()
// 方法参数个数
int getParameterCount()
// 方法参数类型
Class<?>[] getParameterTypes()
// 方法参数注解
Annotation[][] getParameterAnnotations()
// 方法返回值类型
Class<?> getReturnType()
// 方法异常类型
Class<?>[] getExceptionTypes()
// 方法的修饰符，返回值是一个int，不同的 bit 表示不同的含义。使用 Modifier 工具类进行解析
int getModifiers()
```

方法调用：

```java
// 第一个参数为指定的实例对象。
// 如果是静态方法，可传 null。
// 如果是非 public 方法，需先设置 setAccessible(true)
Object invoke(Object obj, Object... args)
```

# 构造方法

创建新实例的几种方式：

```java
// 方式一：通过 new 操作符
Person p = new Person();

// 方式二：通过反射，调用 Class 提供的 newInstance() 方法。局限是只能调用其 public 无参构造方法
Person p = Person.class.newInstance();

// 方式三：通过反射，调用 Constructor 提供的 newInstance() 方法。如果是非 public 方法，需先设置 setAccessible(true)
// 这里通过 private Person(String name) 构造新实例
Constructor<Person> constructor = Person.class.getDeclaredConstructor(String.class);
constructor.setAccessible(true);
Person peter = constructor.newInstance("Peter");
```

# 继承关系

```java
// 获取父类（Object 的父类是 null，其他任何非 interface 的 Class 都必定存在一个父类类型）
Class<? super T> getSuperclass()

// 获取已实现接口（只返回当前类直接实现的接口类型，并不包括其父类实现的接口类型）
Class<?>[] getInterfaces()

// 使用 instanceof 操作符判断一个实例的继承关系
Object n = Integer.valueOf(123);
boolean isDouble = n instanceof Double; // false
boolean isInteger = n instanceof Integer; // true
boolean isNumber = n instanceof Number; // true
boolean isSerializable = n instanceof java.io.Serializable; // true

// 如果 instanceof 为 true，可以使用以下方法做强制类型转换
Number num1 = (Number) n;
Number num2 = Number.class.cast(n);

// 如果是两个 Class 实例，要判断一个向上转型是否成立，可以调用 isAssignableFrom()
Integer.class.isAssignableFrom(Integer.class); // true，因为Integer可以赋值给Integer
Number.class.isAssignableFrom(Integer.class); // true，因为Integer可以赋值给Number
Object.class.isAssignableFrom(Integer.class); // true，因为Integer可以赋值给Object
Integer.class.isAssignableFrom(Number.class); // false，因为Number不能赋值给Integer
```

# 参考

https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/index.html

https://docs.oracle.com/javase/tutorial/reflect/index.html

[JDK1.8 反射包新增了`Parameter`类](https://www.cnblogs.com/Kidezyq/p/11763988.html)

https://www.liaoxuefeng.com/wiki/1252599548343744/1255945147512512

API：

- https://docs.oracle.com/javase/8/docs/api/java/lang/Package.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Field.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html
- https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Parameter.html