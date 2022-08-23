---
title: 函数式编程系列（三）Java 8 Stream API 总结
date: 2019-05-21 20:35:02
updated:
tags: [函数式编程, Java]
typora-root-url: ..
---

流是从支持数据处理操作的源生成的元素序列，源可以是数组、集合、文件、函数。流不是集合元素，它不是数据结构并不保存数据，它的主要目的在于计算。

本文总结下 Stream API：

![java.util.stream](/img/java/lambda/stream_api.png)

# 创建流

## 数组

通过 `Arrays.stream` 方法生成流，并且该方法生成的流是数值流（即 `IntStream` 而不是 `Stream<Integer>`）。使用数值流可以避免计算过程中的拆箱装箱，提高性能。Stream API 提供了 `mapToInt`、`mapToDouble`、`mapToLong` 三种方式将对象流（`Stream<T>`）转换成对应的数值流，同时提供了 `boxed` 方法将数值流转换为对象流：

```java
int[] intArr = new int[]{1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(intArr);

long[] longArr = new long[]{1L, 2L, 3L, 4L, 5L};
LongStream longStream = Arrays.stream(longArr);

double[] doubleArr = new double[]{1.0, 2.0, 3.0, 4.0, 5.0};
DoubleStream doubleStream = Arrays.stream(doubleArr);
```

## 集合

通过集合生成，最常用的一种：

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
Stream<Integer> stream = integerList.stream();
```

## 文件

通过文件生成，得到的每个流是给定文件中的每一行：

```java
// [1, 2, 3, 4, 5]
Stream<String> stream1 = Files.lines(Paths.get("E:\\data.txt"), Charset.defaultCharset());

// [1, 2, 3, 4, 5]
BufferedReader reader = new BufferedReader(new FileReader("E:\\data.txt"));
Stream<String> stream2 = reader.lines();
```

## 函数

```java
// []
Stream<Integer> stream3 = Stream.empty();

// [1, 2, 3, 4, 5]
Stream<Integer> stream4 = Stream.of(1, 2, 3, 4, 5);

// iterate 方法接受两个参数，第一个为初始化值，第二个为进行的函数操作，因为 iterate 生成的流为无限流，因此通过 limit 方法对流进行了截断，只生成 5 个偶数
// [0, 2, 4, 6, 8]
Stream<Integer> stream5 = Stream.iterate(0, n -> n + 2).limit(5);

// generate 方法接受一个参数，方法参数类型为 Supplier<T> ，由它为流提供值。generate 生成的流也是无限流，因此通过 limit 对流进行了截断
// [0.0819448251044178, 0.9273399484995596, 0.3050941986467305, 0.824966110053092, 0.6101914799225238]
Stream<Double> stream6 = Stream.generate(Math::random).limit(5);

// 使用 builder 模式创建流
// [1, 2]
Stream<Integer> stream8 = Stream.<Integer>builder().add(1).add(2).build();

// 使用 concat 方法拼接两个流
// [1, 2, 3, 4, 5]
Stream<Integer> stream7 = Stream.concat(Stream.of(1, 2), Stream.of(3, 4, 5));
```

# 中间操作

一个流可以后面跟随零个或多个中间操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的，仅仅调用到这类方法，并没有真正开始流的遍历，真正的遍历需等到终结操作。

# 终止操作

一个流有且只能有一个终结操作，当这个操作执行后，流就被关闭，无法再被操作了。

## 转换为数组（toArray）

```Java
Object[] objects = Stream.of(1, 2, 3, 4, 5).toArray();
Integer[] integers = Stream.of(1, 2, 3, 4, 5).toArray(Integer[]::new);
int[] arr = IntStream.of(1, 2, 3, 4, 5).toArray();
```

## 转换为列表（toList）

```java
// Java 8, modifiable List
List<String> result = list.stream()
    .collect(Collectors.toList());

// Java 10, unmodifiable List
List<String> result = list.stream()
    .collect(Collectors.toUnmodifiableList());

// Java 16, unmodifiable List
List<String> result = list.stream()
    .toList();
```

## 转换为键值对（toMap）

测试数据如下：

```java
List<Pair<String, Integer>> peoples = Arrays.asList(Pair.of("Lucy", 10),
                                                    Pair.of("Lucy", 30),
                                                    Pair.of("Peter", 18));
```

需求：按 key 分组，key 冲突则保留 value 最大的。

```java
// 演示 `mergeFunction`
// {Peter=(Peter,18), Lucy=(Lucy,30)}
Map<String, Pair<String, Integer>> map = peoples.stream()
  .collect(Collectors.toMap(
    Pair::getKey,
    Function.identity(),
    (people1, people2) -> people1.getValue() > people2.getValue() ? people1 : people2)
  );
```

## 分组统计（groupingBy）

需求：按 key 分组统计总个数、总和、平均数、最大值、最小值。

方式一，各项单独统计：

```java
// {Lucy=2, Peter=1}
Map<String, Long> counting = peoples.stream()
    .collect(Collectors.groupingBy(Pair::getKey, Collectors.counting()));
// {Lucy=40, Peter=18}
Map<String, Integer> summing = peoples.stream()
    .collect(Collectors.groupingBy(Pair::getKey, Collectors.summingInt(Pair::getValue)));
// {Lucy=20.0, Peter=18.0}
Map<String, Double> averaging = peoples.stream()
    .collect(Collectors.groupingBy(Pair::getKey, Collectors.averagingDouble(Pair::getValue)));
// {Lucy=Optional[(Lucy,30)], Peter=Optional[(Peter,18)]}
Map<String, Optional<Pair<String, Integer>>> max = peoples.stream()
    .collect(Collectors.groupingBy(Pair::getKey, Collectors.maxBy(Comparator.comparing(Pair::getValue))));
// {Lucy=Optional[(Lucy,10)], Peter=Optional[(Peter,18)]}
Map<String, Optional<Pair<String, Integer>>> min = peoples.stream()
    .collect(Collectors.groupingBy(Pair::getKey, Collectors.minBy(Comparator.comparing(Pair::getValue))));
```

方式二，汇总统计：

```java
// {
//   Lucy=IntSummaryStatistics{count=2, sum=40, min=10, average=20.000000, max=30}, 
//   Peter=IntSummaryStatistics{count=1, sum=18, min=18, average=18.000000, max=18}
// }
Map<String, IntSummaryStatistics> summary = peoples.stream()
    .collect(Collectors.groupingBy(Pair::getKey, Collectors.summarizingInt(Pair::getValue)));
```

参考：https://www.baeldung.com/java-groupingby-collector

# 常见问题

## 获取列表索引

`forEach` 方法入参缺少列表索引，无法实现某些特殊需求。

解决方案一，通过 `IntStream` 获取索引 index：

```Java
IntStream.range(0, elements.size())
        .forEach(index -> downloadFile(elements.get(index), index));
```

解决方案二，自定义工具类通过 `BiConsumer` 传参，获取索引 index 和元素 element：

```Java
public class IterateUtil {
    public static <E> void forEach(
            Iterable<? extends E> elements, BiConsumer<Integer, ? super E> action) {
        Objects.requireNonNull(elements);
        Objects.requireNonNull(action);

        int index = 0;
        for (E element : elements) {
            action.accept(index++, element);
        }
    }
}

// 使用方式
IterateUtil.forEach(
    elements, 
    (index, element) -> downloadFile(element, index)
);
```

## 异常处理

《[Exceptions in Java 8 Lambda Expressions](https://www.baeldung.com/java-lambda-exceptions)》

《[Stream 中异常处理的四种方式](https://www.jianshu.com/p/597a7ccfec25)》

# 参考

https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html

https://docs.oracle.com/javase/tutorial/collections/streams/index.html

https://github.com/amaembo/streamex

https://www.baeldung.com/tag/java-streams/

* [Merging Two Maps with Java 8](https://www.baeldung.com/java-merge-maps)

《[用了 Stream API 之后，代码反而越写越丑？——写出具有可维护性的 Stream API 代码](https://mp.weixin.qq.com/s/a_QYX5z1AJhITYaXD-Gzag)》

