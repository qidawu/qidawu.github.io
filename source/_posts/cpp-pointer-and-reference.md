---
title: C/C++ 语言系列（四）复合数据类型之指针、引用
date: 2021-02-06 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# 引用

引用是一个别名，也就是说，它是某个已存在变量的另一个名字。修改引用等同于修改被引用变量自身。

一个变量可以有多个引用。

## 声明引用

不存在**空引用**。声明引用的同时，必须初始化，否则报编译错误如下：

```C
int &ref;  // declaration of reference variable 'ref' requires an initializer
```

引用一旦初始化，就不能再指向另一个变量。修改引用等同于修改被引用变量本身。

# 指针

指针是一个变量，其值为另一个变量的地址，即，内存位置的直接地址。

## 声明指针

一元运算符 `*` 用于声明一个指针变量：

```C
  bool   *bp;    // 声明一个布尔型的空指针
  char   *ch;    // 声明一个字符型的空指针
  int    *ip;    // 声明一个整型的空指针
  double *dp;    // 声明声明一个 double 型的空指针
  float  *fp;    // 声明一个浮点型的空指针
```

* 可以声明空指针。
* 除了常指针，其它指针可以在任何时间被初始化。
* 所有指针的值的实际数据类型，不管是布尔型、字符型、整型、浮点型，还是其它的数据类型，都是一样的，其值都是一个**代表内存地址**的**十六进制数**。
* 不同数据类型的指针之间唯一的不同是，指针所指向的变量或常量的数据类型不同，**因此执行递增或递减时的步长不同**。

## 获取指针大小

| 操作系统 | 指针变量的存储大小 |
| -------- | ------------------ |
| 32 bits  | 4 Bytes            |
| 64 bits  | 8 Bytes            |

本机为 64 位操作系统，验证如下：

```C
sizeof(bool*)  // 8
sizeof(char*)  // 8
sizeof(int*)    // 8
sizeof(float*)  // 8
sizeof(double*) // 8
```

## 指针的算数运算

可以对指针进行四种算术运算：`++`、`--`、`+`、`-`。运算后，指针保存新的地址。

```C
  int a = 0;
  int b = 1;
  int * p = &a;

  p = &b;
  p += 1;
  p -= 1;
  p++;
  p--;
  ++p;
  --p;
```

## 重点考点

```C
// 常指针
int* const p = &a;

// 指向常量的指针
const int * p;

// 指向常量的常指针
const int * const p = &a;
```

### 常指针

顾名思义，指针本身是个常量，不能修改。

```C
  int a = 0;
  int b = 1;
  int* const p = &a;

  // 表达式必须是可修改的左值
  // cannot assign to variable 'p' with const-qualified type 'int *const'
  p = &b;
  p += 1;
  p -= 1;
  p++;
  p--;
  ++p;
  --p;
```

常见的常指针，例如：

* 数组名

### 指向常量的指针

顾名思义，指针指向的是常量，不能修改其值。

```C
  int a = 0;
  int b = 1;
  const int * p = &a;

  // 指针本身可以重新赋值
  p = &b;

  // 但不能修改指针指向的常量的值，否则报编译错误：
  // 表达式必须是可修改的左值
  // read-only variable is not assignable
  *p += 1;
  *p -= 1;
  (*p)++;
  (*p)--;
  ++(*p);
  --(*p);
```

### 指向常量的常指针

结合了上述两种特性。

## 常见指针

### 字符串指针

```C++
// 字符串指针（指向一个常量字符串，它被存放在程序的静态数据区，一旦定义就不能改变）
char * str = "hello world";
```

### 数组指针

```C++
int arrayName[5] = {1, 2, 3, 4, 5};  // &arrayName = 0x7ffee7376620
int * p = arrayName;  // // 数组指针（Pointers to array，即指向数组中第一个元素的地址）

for (int i = 0; i < 5; i++)
{
  cout << p << endl;  // 0x7ffee7376620 0x7ffee7376624 0x7ffee7376628 0x7ffee737662c 0x7ffee7376630
  cout << *p++ << endl;  // 1 2 3 4 5
}
```

### 结构体指针

```C++
product apple;
// 结构体指针（Pointers to struct）
product * p = &apple;
// The arrow operator (->) is a dereference operator that is used exclusively with pointers to objects that have members. This operator serves to access the member of an object directly from its address.
p->weight;
// equivalent to:
(*p).weight;
```

### 类指针

```C++
Rectangle rect(3, 4);
// 类指针（Pointers to classes），主要用于多态性
Shape * p = &rect;

// member y of object x
rect.area();  // 12
// member y of object pointed to by x
p->area();    // 12
// equivalent to:
(*p).area();  // 12
```

# 引用与指针对比

常见问题，下列代码的区别？


```C
*p
&p
*&p
&*p
```

要解决这个问题，首先需要了解这两个运算符的区别：

|      | 在赋值运算符左侧 | 在赋值运算符右侧   |
| ---- | ---------------- | ------------------ |
| `*`  | 表示声明指针     | 表示**取值运算符** |
| `&`  | 表示声明引用     | 表示**取址运算符** |

代码示例如下：

```C
  int a = 10;
  int& b = a;
  int* p = &a;

  cout << "a = " << a << ", &a = " << &a << ", *&a = " << *&a << endl;
  cout << "b = " << b << ", &b = " << &b << ", *&b = " << *&b << endl;
  cout << "p = " << p << ", &p = " << &p << ", *p = " << *p << ", &*p = " << &*p << ", *&p = " << *&p << endl
```

输出结果如下：

```
a = 10, &a = 0x7ffee483763c, *&a = 10
b = 10, &b = 0x7ffee483763c, *&b = 10
p = 0x7ffee483763c, &p = 0x7ffee4837628, *p = 10, &*p = 0x7ffee483763c, *&p = 0x7ffee483763c
```

总结如下：

| 指针变量 | 结果           | 描述                                      |
| -------- | -------------- | ----------------------------------------- |
| `p`      | 0x7ffee483763c | 返回指针变量 `p` 保存的地址               |
| `*p`     | 10             | 返回指针变量 `p` 保存的地址的实际值       |
| `&p`     | 0x7ffee4837628 | 返回指针变量 `p` 自身的地址               |
| `&*p`    | 0x7ffee483763c | 返回指针变量 `p` 保存的地址的实际值的地址 |
| `*&p`    | 0x7ffee483763c | 返回指针变量 `p` 自身的地址的实际值       |

引用与指针的区别，如下图：

![引用与指针对比](/img/c/pointer_and_reference.png)

# 参考

http://www.cplusplus.com/doc/tutorial/pointers/

[Difference between pointer and reference in C ?](https://stackoverflow.com/questions/4995899/difference-between-pointer-and-reference-in-c)

[C++ 中的参数传递方式：传值、传地址、传引用总结](https://cdlwhm1217096231.github.io/C/C-%E4%B8%AD%E7%9A%84%E5%8F%82%E6%95%B0%E4%BC%A0%E9%80%92%E6%96%B9%E5%BC%8F%EF%BC%9A%E4%BC%A0%E5%80%BC%E3%80%81%E4%BC%A0%E5%9C%B0%E5%9D%80%E3%80%81%E4%BC%A0%E5%BC%95%E7%94%A8%E6%80%BB%E7%BB%93/)

[32/64 位系统支持多大内存？](https://www.crucial.cn/learn-with-crucial/memory/how-much-memory-does-your-windows-support)

