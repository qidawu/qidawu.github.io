---
title: Java 集合框架系列（六）线性表之 List 总结
date: 2018-05-11 22:40:10
updated:
tags: [Java, 数据结构]
typora-root-url: ..
---

# 常用方法

## List partition

《[集合 List 分片的 5 种方法](https://mp.weixin.qq.com/s/PgVr5TldcDyW28clz6bF7g)》

## Array to List

方法一：转两次

```java
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"))  // from varargs
```

方法二：Java 8 

```java
// 包装类型
Integer [] myArray = { 1, 2, 3 };
List<Integer> myList = Arrays.stream(myArray).collect(Collectors.toList());
// 基本类型也可以实现转换（依赖 boxed 的装箱操作）
int [] myArray2 = { 1, 2, 3 };
List<Integer> myList2 = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```

方法三：Guava

```java
// 不可变集合
List<String> il = ImmutableList.of("string", "elements");  // from varargs
List<String> i2 = ImmutableList.copyOf(new String[]{"string", "elements"});  // from array

// 可变集合
List<String> l0 = Lists.newLinkedList(Arrays.asList("a", "b", "c"));  // from collection
List<String> l1 = Lists.newArrayList(Arrays.asList("a", "b", "c"));  // from collection
List<String> l2 = Lists.newArrayList(new String[]{"string", "elements"});  // from array
List<String> l3 = Lists.newArrayList("or", "string", "elements"); // from varargs
```

# 参考

《[Arrays.asList() 原来是这样用的](https://mp.weixin.qq.com/s/-jWmAqAbc-vLV9w541NhxQ)》

《[千万不要这样使用 Arrays.asList !](https://mp.weixin.qq.com/s/7ILvz9UreVco0sq9s-zJNg)》
