---
title: Java 常用类型系列（一）包装类型总结
date: 2018-04-01 21:28:04
updated:
tags: Java
typora-root-url: ..
---

包装类型作为日常开发最常用的数据载体，使用时有一些点需要特别注意，否则容易踩坑，本文总结下。

# 值、引用比较问题

首先看一段代码：

```java
Integer a = 1000;
Integer b = 1000;
Integer c = 100;
Integer d = 100;
Integer e = new Integer(100);
Integer f = new Integer(100);

// 值比较
System.out.println(("a equals b is " + (a.equals(b))));  // a equals b is true

// 引用比较，结果为 false
System.out.println("a == b is " + (a == b));  // a == b is false
// 引用比较，结果意外的为 true。这是由于 Integer 自动装箱时 [-128, 127] 使用了缓存，详见其 valueOf 方法的源码实现。
System.out.println(("c == d is " + (c == d)));  // c == d is true
// 引用比较，结果为 false。因为 new Integer(int) 构造方法没有使用缓存。
System.out.println(("e == f is " + (e == f)));  // e == f is false
```

值的比较务必使用 `equals` 方法。可以遵循《阿里巴巴 Java 开发规范》这条规范：

![notes](/img/java/primitive-type/notes.png)

# 自动装箱的性能问题

Java 语言提供了八种基本类型。其中包括六种数字类型（四个整数型，两个浮点型），一种字符类型，还有一种布尔型。这些基本类型（primitive type）都有对应的包装类型（boxed primitive type），具有类的特性。基本类型和包装类型之间的转换通过装箱和拆箱方法：

| 基本数据类型 | 对应的包装类 | 拆箱方法         | 装箱方法           | 存储空间 | 取值范围     | 缓存范围                |
| ------------ | ------------ | ---------------- | ------------------ | -------- | ------------ | ----------------------- |
| `byte`       | `Byte`       | `byteValue()`    | `valueOf(byte)`    | 8 bit    | -2^7~2^7-1   | -2^7~2^7-1              |
| `short`      | `Short`      | `shortValue()`   | `valueOf(short)`   | 16 bit   | -2^15~2^15-1 | -2^7~2^7-1              |
| `int`        | `Integer`    | `intValue()`     | `valueOf(int)`     | 32 bit   | -2^31~2^31-1 | -2^7~2^7-1              |
| `long`       | `Long`       | `longValue()`    | `valueOf(long)`    | 64 bit   | -2^63~2^63-1 | -2^7~2^7-1              |
| `float`      | `Float`      | `floatValue()`   | `valueOf(float)`   | 32 bit   |              |                         |
| `double`     | `Double`     | `doubleValue()`  | `valueOf(double)`  | 64 bit   |              |                         |
| `boolean`    | `Boolean`    | `booleanValue()` | `valueOf(boolean)` | 1 bit    |              | true & false            |
| `char`       | `Character`  | `charValue()`    | `valueOf(char)`    | 16 bit   |              | \u005Cu0000~\u005Cu007F |

其中数字类型的拆箱方法由共同的父类 `java.lang.Number` 定义：

![Number方法](/img/java/primitive-type/Number方法.png)

![Number](/img/java/primitive-type/Number.png)

自 JDK 1.5 版本后，Java 引入了自动装箱（autoboxing）、拆箱（auto-unboxing）作为语法糖，使基本类型和包装类型可以直接转换，减少使用包装类型的繁琐性。`javac` 编译会做相应处理，例如：

```java
public static void main(String[] args) {
    Integer i = 10;
    int n = i;
}
```

反编译后代码如下：

```java
public static void main(String args[]) {
    Integer i = Integer.valueOf(10);  // 自动装箱
    int n = i.intValue();  // 自动拆箱
}
```

但在进行大批量数据操作时，装箱操作会有一定的性能损耗，看一段代码：

```java
// 程序一：包装类型
Long sum = 0L;
for (long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i;
}
System.out.println(sum);  // 耗时 10390 ms

// 程序二：基本类型
long sum = 0L;
for (long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i;
}
System.out.println(sum);  // 耗时 1101 ms
```

程序一运行起来比程序二慢了十倍，因为变量 `sum` 的每次计算结果都被反复装箱，导致不必要的对象创建和较高的资源消耗，从而影响性能。代码反编译如下：

```java
// 程序一：包装类型
Long sum = Long.valueOf(0L);
for (long i = 0L; i < 2147483647L; i++) {
    sum = Long.valueOf(sum.longValue() + i);
}
System.out.println(sum);

// 程序二：基本类型
long sum = 0L;
long i;
for (i = 0L; i < 2147483647L; i++) {
    sum += i;
}
System.out.println(sum);
```

这提醒我们，在进行大批量数据操作时，要优先使用基本类型。为此 Java 8 还专门引入了：

* `IntStream`、`LongStream`、`DoubleStream` 作为泛型类 `Stream` 的补充；
* `OptionalInt`、`OptionalLong`、`OptionalDouble` 作为泛型类 `Optional` 的补充；

* 以及一堆配套的基本类型特化的函数式接口。

部分代码如下：

![Stream](/img/java/lambda/Stream.png)

![stream_methods](/img/java/lambda/stream_methods.png)

那么什么时候应该使用装箱类型呢？

1. 必须使用装箱基本类型作为类型参数，因为 Java 不允许使用基本类型。例如作为泛型集合中的元素、键和值，由于不能将基本类型作为类型参数，因此必须使用装箱基本类型。又例如，不能将变量声明为 `ThreadLocal<int>` 类型，因此必须使用 `ThreadLocal<Integer>`。
2. 在进行反射的方法调用时，必须使用装箱基本类型。

# 自动拆箱的 NPE 风险

要注意，当程序进行涉及装箱和拆箱基本类型的混合类型计算时，它会进行自动拆箱。当程序进行拆箱时，会有 NPE 风险：

```java
Long sum = null;
for (long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i;  // java.lang.NullPointerException
}
System.out.println(sum);
```

在日常开发中，数据库字段定义、或查询结果都可能为 `null`，因为自动拆箱，用基本类型接收会有 NPE 风险。可以遵循《阿里巴巴 Java 开发规范》这条规范：

![notes2](/img/java/primitive-type/notes2.png)

# 进制转换

|          | 英文简写 | 字面量前缀 | 备注                                                         |
| -------- | -------- | ---------- | ------------------------------------------------------------ |
| 十六进制 | `HEX`    | `0X`、`0x` |                                                              |
| 十进制   | `DEC`    | 无         |                                                              |
| 八进制   | `OCT`    | `0`        |                                                              |
| 二进制   | `BIN`    | `0B`、`0b` | 参考[官方文档](https://docs.oracle.com/javase/7/docs/technotes/guides/language/binary-literals.html) |

```java
// 十进制转成二进制：11111111
Integer.toBinaryString(255);
// 二进制转十进制：255
Integer.valueOf("11111111", 2);

// 十进制转成八进制：10
Integer.toOctalString(8);
// 八进制转成十进制：8
Integer.valueOf("10", 8);

// 十进制转成十六进制：F
Integer.toHexString(15);
// 十六进制转成十进制：15
Integer.valueOf("F", 16);
```

# 源码解析

最后，来总结下包装类型特点：

* 所有包装类型都为 `final`，底层被包装的基本类型也都为 `final`。因此包装类型一旦初始化后，在运行期值是不可变的，非常安全。
* `Byte`、`Short`、`Integer`、`Long` 装箱方法的某个数据段使用了缓存，因此 `==` 引用比较时可能会出现预期外的结果（`true`），详见源码。

## java.lang.Byte

```java
public final class Byte extends Number implements Comparable<Byte> {

    /**
     * The value of the {@code Byte}.
     *
     * @serial
     */
    private final byte value;

    /**
     * Returns a {@code Byte} instance representing the specified
     * {@code byte} value.
     * If a new {@code Byte} instance is not required, this method
     * should generally be used in preference to the constructor
     * {@link #Byte(byte)}, as this method is likely to yield
     * significantly better space and time performance since
     * all byte values are cached.
     *
     * @param  b a byte value.
     * @return a {@code Byte} instance representing {@code b}.
     * @since  1.5
     */
    public static Byte valueOf(byte b) {
        final int offset = 128;
        return ByteCache.cache[(int)b + offset];
    }

    /**
     * Returns the value of this {@code Byte} as a
     * {@code byte}.
     */
    public byte byteValue() {
        return value;
    }

    /**
     * Compares this object to the specified object.  The result is
     * {@code true} if and only if the argument is not
     * {@code null} and is a {@code Byte} object that
     * contains the same {@code byte} value as this object.
     *
     * @param obj       the object to compare with
     * @return          {@code true} if the objects are the same;
     *                  {@code false} otherwise.
     */
    public boolean equals(Object obj) {
        if (obj instanceof Byte) {
            return value == ((Byte)obj).byteValue();
        }
        return false;
    }
}
```

## java.lang.Short

```java
public final class Short extends Number implements Comparable<Short> {

    /**
     * The value of the {@code Short}.
     *
     * @serial
     */
    private final short value;

    /**
     * Returns a {@code Short} instance representing the specified
     * {@code short} value.
     * If a new {@code Short} instance is not required, this method
     * should generally be used in preference to the constructor
     * {@link #Short(short)}, as this method is likely to yield
     * significantly better space and time performance by caching
     * frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  s a short value.
     * @return a {@code Short} instance representing {@code s}.
     * @since  1.5
     */
    public static Short valueOf(short s) {
        final int offset = 128;
        int sAsInt = s;
        if (sAsInt >= -128 && sAsInt <= 127) { // must cache
            return ShortCache.cache[sAsInt + offset];
        }
        return new Short(s);
    }

    /**
     * Returns the value of this {@code Short} as a
     * {@code short}.
     */
    public short shortValue() {
        return value;
    }

    /**
     * Compares this object to the specified object.  The result is
     * {@code true} if and only if the argument is not
     * {@code null} and is a {@code Short} object that
     * contains the same {@code short} value as this object.
     *
     * @param obj       the object to compare with
     * @return          {@code true} if the objects are the same;
     *                  {@code false} otherwise.
     */
    public boolean equals(Object obj) {
        if (obj instanceof Short) {
            return value == ((Short)obj).shortValue();
        }
        return false;
    }

}
```

## java.lang.Integer

```java
public final class Integer extends Number implements Comparable<Integer> {

    /**
     * The value of the {@code Integer}.
     *
     * @serial
     */
    private final int value;

    /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }

    /**
     * Returns the value of this {@code Integer} as an
     * {@code int}.
     */
    public int intValue() {
        return value;
    }

    /**
     * Compares this object to the specified object.  The result is
     * {@code true} if and only if the argument is not
     * {@code null} and is an {@code Integer} object that
     * contains the same {@code int} value as this object.
     *
     * @param   obj   the object to compare with.
     * @return  {@code true} if the objects are the same;
     *          {@code false} otherwise.
     */
    public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }

}
```

## java.lang.Long

```java
public final class Long extends Number implements Comparable<Long> {

    /**
     * The value of the {@code Long}.
     *
     * @serial
     */
    private final long value;

    /**
     * Returns a {@code Long} instance representing the specified
     * {@code long} value.
     * If a new {@code Long} instance is not required, this method
     * should generally be used in preference to the constructor
     * {@link #Long(long)}, as this method is likely to yield
     * significantly better space and time performance by caching
     * frequently requested values.
     *
     * Note that unlike the {@linkplain valueOf(int)
     * corresponding method} in the {@code Integer} class, this method
     * is <em>not</em> required to cache values within a particular
     * range.
     *
     * @param  l a long value.
     * @return a {@code Long} instance representing {@code l}.
     * @since  1.5
     */
    public static Long valueOf(long l) {
        final int offset = 128;
        if (l >= -128 && l <= 127) { // will cache
            return LongCache.cache[(int)l + offset];
        }
        return new Long(l);
    }

    /**
     * Returns the value of this {@code Long} as a
     * {@code long} value.
     */
    public long longValue() {
        return value;
    }

    /**
     * Compares this object to the specified object.  The result is
     * {@code true} if and only if the argument is not
     * {@code null} and is a {@code Long} object that
     * contains the same {@code long} value as this object.
     *
     * @param   obj   the object to compare with.
     * @return  {@code true} if the objects are the same;
     *          {@code false} otherwise.
     */
    public boolean equals(Object obj) {
        if (obj instanceof Long) {
            return value == ((Long)obj).longValue();
        }
        return false;
    }

}
```

## java.lang.Float

```java
public final class Float extends Number implements Comparable<Float> {

    /**
     * The value of the Float.
     *
     * @serial
     */
    private final float value;

    /**
     * Returns a {@code Float} instance representing the specified
     * {@code float} value.
     * If a new {@code Float} instance is not required, this method
     * should generally be used in preference to the constructor
     * {@link #Float(float)}, as this method is likely to yield
     * significantly better space and time performance by caching
     * frequently requested values.
     *
     * @param  f a float value.
     * @return a {@code Float} instance representing {@code f}.
     * @since  1.5
     */
    public static Float valueOf(float f) {
        return new Float(f);
    }

    /**

     * Compares this object against the specified object.  The result
     * is {@code true} if and only if the argument is not
     * {@code null} and is a {@code Float} object that
     * represents a {@code float} with the same value as the
     * {@code float} represented by this object. For this
     * purpose, two {@code float} values are considered to be the
     * same if and only if the method {@link #floatToIntBits(float)}
     * returns the identical {@code int} value when applied to
     * each.
     *
     * <p>Note that in most cases, for two instances of class
     * {@code Float}, {@code f1} and {@code f2}, the value
     * of {@code f1.equals(f2)} is {@code true} if and only if
     *
     * <blockquote><pre>
     *   f1.floatValue() == f2.floatValue()
     * </pre></blockquote>
     *
     * <p>also has the value {@code true}. However, there are two exceptions:
     * <ul>
     * <li>If {@code f1} and {@code f2} both represent
     *     {@code Float.NaN}, then the {@code equals} method returns
     *     {@code true}, even though {@code Float.NaN==Float.NaN}
     *     has the value {@code false}.
     * <li>If {@code f1} represents {@code +0.0f} while
     *     {@code f2} represents {@code -0.0f}, or vice
     *     versa, the {@code equal} test has the value
     *     {@code false}, even though {@code 0.0f==-0.0f}
     *     has the value {@code true}.
     * </ul>
     *
     * This definition allows hash tables to operate properly.
     *
     * @param obj the object to be compared
     * @return  {@code true} if the objects are the same;
     *          {@code false} otherwise.
     * @see java.lang.floatToIntBits(float)
     */
    public boolean equals(Object obj) {
        return (obj instanceof Float)
               && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
    }

}
```

## java.lang.Double

```java
public final class Double extends Number implements Comparable<Double> {

    /**
     * The value of the Double.
     *
     * @serial
     */
    private final double value;

    /**
     * Returns a {@code Double} instance representing the specified
     * {@code double} value.
     * If a new {@code Double} instance is not required, this method
     * should generally be used in preference to the constructor
     * {@link #Double(double)}, as this method is likely to yield
     * significantly better space and time performance by caching
     * frequently requested values.
     *
     * @param  d a double value.
     * @return a {@code Double} instance representing {@code d}.
     * @since  1.5
     */
    public static Double valueOf(double d) {
        return new Double(d);
    }

    /**
     * Compares this object against the specified object.  The result
     * is {@code true} if and only if the argument is not
     * {@code null} and is a {@code Double} object that
     * represents a {@code double} that has the same value as the
     * {@code double} represented by this object. For this
     * purpose, two {@code double} values are considered to be
     * the same if and only if the method {@link
     * #doubleToLongBits(double)} returns the identical
     * {@code long} value when applied to each.
     *
     * <p>Note that in most cases, for two instances of class
     * {@code Double}, {@code d1} and {@code d2}, the
     * value of {@code d1.equals(d2)} is {@code true} if and
     * only if
     *
     * <blockquote>
     *  {@code d1.doubleValue() == d2.doubleValue()}
     * </blockquote>
     *
     * <p>also has the value {@code true}. However, there are two
     * exceptions:
     * <ul>
     * <li>If {@code d1} and {@code d2} both represent
     *     {@code Double.NaN}, then the {@code equals} method
     *     returns {@code true}, even though
     *     {@code Double.NaN==Double.NaN} has the value
     *     {@code false}.
     * <li>If {@code d1} represents {@code +0.0} while
     *     {@code d2} represents {@code -0.0}, or vice versa,
     *     the {@code equal} test has the value {@code false},
     *     even though {@code +0.0==-0.0} has the value {@code true}.
     * </ul>
     * This definition allows hash tables to operate properly.
     * @param   obj   the object to compare with.
     * @return  {@code true} if the objects are the same;
     *          {@code false} otherwise.
     * @see java.lang.Double#doubleToLongBits(double)
     */
    public boolean equals(Object obj) {
        return (obj instanceof Double)
               && (doubleToLongBits(((Double)obj).value) ==
                      doubleToLongBits(value));
    }

}
```

## java.lang.Boolean

```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean> {

    /**
     * The value of the Boolean.
     *
     * @serial
     */
    private final boolean value;

    /**
     * Returns a {@code Boolean} instance representing the specified
     * {@code boolean} value.  If the specified {@code boolean} value
     * is {@code true}, this method returns {@code Boolean.TRUE};
     * if it is {@code false}, this method returns {@code Boolean.FALSE}.
     * If a new {@code Boolean} instance is not required, this method
     * should generally be used in preference to the constructor
     * {@link #Boolean(boolean)}, as this method is likely to yield
     * significantly better space and time performance.
     *
     * @param  b a boolean value.
     * @return a {@code Boolean} instance representing {@code b}.
     * @since  1.4
     */
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }

    /**
     * Returns the value of this {@code Boolean} object as a boolean
     * primitive.
     *
     * @return  the primitive {@code boolean} value of this object.
     */
    public boolean booleanValue() {
        return value;
    }

   /**
     * Returns {@code true} if and only if the argument is not
     * {@code null} and is a {@code Boolean} object that
     * represents the same {@code boolean} value as this object.
     *
     * @param   obj   the object to compare with.
     * @return  {@code true} if the Boolean objects represent the
     *          same value; {@code false} otherwise.
     */
    public boolean equals(Object obj) {
        if (obj instanceof Boolean) {
            return value == ((Boolean)obj).booleanValue();
        }
        return false;
    }

}
```

## java.lang.Character

```java
public final class Character implements java.io.Serializable, Comparable<Character> {

    /**
     * The value of the {@code Character}.
     *
     * @serial
     */
    private final char value;

    /**
     * Returns a <tt>Character</tt> instance representing the specified
     * <tt>char</tt> value.
     * If a new <tt>Character</tt> instance is not required, this method
     * should generally be used in preference to the constructor
     * {@link #Character(char)}, as this method is likely to yield
     * significantly better space and time performance by caching
     * frequently requested values.
     *
     * This method will always cache values in the range {@code
     * '\u005Cu0000'} to {@code '\u005Cu007F'}, inclusive, and may
     * cache other values outside of this range.
     *
     * @param  c a char value.
     * @return a <tt>Character</tt> instance representing <tt>c</tt>.
     * @since  1.5
     */
    public static Character valueOf(char c) {
        if (c <= 127) { // must cache
            return CharacterCache.cache[(int)c];
        }
        return new Character(c);
    }

    /**
     * Returns the value of this {@code Character} object.
     * @return  the primitive {@code char} value represented by
     *          this object.
     */
    public char charValue() {
        return value;
    }

    /**
     * Compares this object against the specified object.
     * The result is {@code true} if and only if the argument is not
     * {@code null} and is a {@code Character} object that
     * represents the same {@code char} value as this object.
     *
     * @param   obj   the object to compare with.
     * @return  {@code true} if the objects are the same;
     *          {@code false} otherwise.
     */
    public boolean equals(Object obj) {
        if (obj instanceof Character) {
            return value == ((Character)obj).charValue();
        }
        return false;
    }

}
```

# 参考

《Effective Java 第三版》

* 第 44 条：坚持使用标准的函数接口
* 第 61 条：基本类型优先于装箱基本类型

《Java 8 函数式编程》

《Java 8 实战》

https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html

[一文读懂什么是Java中的自动拆装箱](https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650121987&idx=1&sn=70bba3f7f42a269eeada9cecb34c10a5&chksm=f36bba22c41c3334b92f184e402f2f77e2d5e0a2e0a21ec5a8229773afcab4f026a0b6f47fc4&scene=21)