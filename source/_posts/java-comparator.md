---
title: Java 集合框架系列（五）Comparator 排序
date: 2019-05-01 23:46:35
updated:
tags: Java
typora-root-url: ..
---

本文总结下集合排序的常用方法：

![java.util.Comparator](/img/java/collection/java.util.Comparator.png)

# 基于可比较对象排序

如果集合元素已实现 `Comparable` 接口，可以直接使用 `naturalOrder`、`reverseOrder` 方法进行排序：

```java
List<Integer> integers = Arrays.asList(3, 1, 2, 4);

// 升序
// [1, 2, 3, 4]
integers.sort(Comparator.naturalOrder());

// 降序
// [4, 3, 2, 1]
integers.sort(Comparator.reverseOrder());
```

上述排序不支持 `null` 值（会抛 NPE 异常），如果自定义实现的话，代码比较冗余，容易出错：

```java
List<Integer> integers = Arrays.asList(3, 1, null, 2, 4);

// [null, 1, 2, 3, 4]
integers.sort((o1, o2) -> {
    // 写法1：
    if (o1 != null && o2 != null) {
        return o1.compareTo(o2);
    } else {
        return o1 == null ? -1 : 1;
    }
    // 写法2：
    // return o1 == null ? -1 : (o2 == null ? 1 : o1.compareTo(o2));
});

// [4, 3, 2, 1, null]
integers.sort((o1, o2) -> {
    // 写法1：
    if (o1 != null && o2 != null) {
        return o2.compareTo(o1);
    } else {
        return o1 == null ? 1 : -1;
    }
    // 写法2：
    // return o1 == null ? 1 : (o2 == null ? -1 : o2.compareTo(o1));
});
```

可采用 `nullsFirst`、`nullsLast` 方法兼容 `null` 值情况：

```java
List<Integer> integers = Arrays.asList(3, 1, null, 2, 4);

// null 值在前
// [null, 1, 2, 3, 4]
integers.sort(Comparator.nullsFirst(Comparator.naturalOrder()));

// null 值在后
// [4, 3, 2, 1, null]
integers.sort(Comparator.nullsLast(Comparator.reverseOrder()));
```

# 基于不可比较对象排序

例子：

```java
@Data
@AllArgsConstructor
public class IdName {
    private Integer id;
    private String name;
}
```

如果集合元素未实现 `Comparable` 接口，需要抽取关键字（关键字需实现 `Comparable` 接口）排序：

```java
List<IdName> idNames = Arrays.asList(
    new IdName(3, "Pete"),
    new IdName(1, "Tom"),
    new IdName(2, "Ben"),
    new IdName(2, "Allen"));

// 根据 ID 升序
// [IdName(id=1, name=Tom), IdName(id=2, name=Ben), IdName(id=2, name=Allen), IdName(id=3, name=Pete)]
idNames.sort(Comparator.comparing(IdName::getId));

// 根据 ID、Name 复合排序（升序）
// [IdName(id=1, name=Tom), IdName(id=2, name=Allen), IdName(id=2, name=Ben), IdName(id=3, name=Pete)]
idNames.sort(Comparator.comparing(IdName::getId)
             .thenComparing(IdName::getName));

// 根据 ID、Name 复合排序（降序）
// [IdName(id=3, name=Pete), IdName(id=2, name=Ben), IdName(id=2, name=Allen), IdName(id=1, name=Tom)]
idNames.sort(Comparator.comparing(IdName::getId)
             .thenComparing(IdName::getName)
             .reversed());
```

# 参考

《[java comparator 升序、降序、倒序从源码角度理解](https://blog.csdn.net/u013066244/article/details/78997869)》

https://blog.csdn.net/weixin_44270183/article/details/87026995