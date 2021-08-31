---
title: C/C++ 语言系列（八）复合数据类型之类型别名
date: 2021-02-18 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

> A type alias is a different name by which a type can be identified. In C++, any valid type can be aliased so that it can be referred to with a different identifier.

在 C++ 中，有两种创建类型别名的语法：

1. 从 C 语言继承而来，使用 `typedef` 关键字：

   ```C
   typedef existing_type new_type_name ;
   ```

2. 由 C++ 语言引入，使用 `using` 关键字：

   ```C
   using new_type_name = existing_type ;
   ```

`existing_type` 可以是任何类型，无论是基本类型还是复合类型：

例子一：

| `typedef`                    | `using`                      |
| ---------------------------- | ---------------------------- |
| `typedef char C;`            | `using C = char;`            |
| `typedef unsigned int WORD;` | `using WORD = unsigned int;` |
| `typedef char * pChar;`      | `using pChar = char *;`      |
| `typedef char field [50]; `  | `using field = char [50]; `  |

例子二，下面两种定义结构体类型的方式是等价的：

```C++
struct product {
  int weight;
  double price;
};

typedef struct  {
  int weight;
  double price;
} product;
```

`new_type_name` 作为该类型的别名，用法如下：

```C
C mychar, anotherchar, *ptc1;
WORD myword;
pChar ptc2;
field name; 
```

一旦定义了这些别名，就可以像其它有效类型一样，在任何声明中使用。尤其常见于与结构体搭配使用。

# 参考

https://www.cplusplus.com/doc/tutorial/other_data_types/