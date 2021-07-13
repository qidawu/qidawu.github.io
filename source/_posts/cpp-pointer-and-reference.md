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

![引用与指针对比](/img/cpp/pointer_and_reference.png)



# 动态内存

## 动态内存分配

在 C++ 中，您可以使用特殊的运算符为给定类型的变量在运行时分配**堆内存（memory heap）**，这会返回所分配的空间**地址**。

动态内存分配使用 `new` 运算符，后面跟上一个数据类型，语法如下：

```C++
// allocate memory to contain one single element of specified type
pointer = new type
  
// allocate a block (an array) of elements of specified type, where `number_of_elements` is an integer value representing the amount of these.
// it returns a pointer to the beginning of the new block of memory allocated.
pointer = new type [number_of_elements]
```

例如：

```C
int * bar = new int(5); // int bar = 5
int * foo = new int[5]; // int foo[5]
```

![](/img/cpp/dynamic_memory.png)

> In this case, the system dynamically allocates space for five elements of type `int` and returns a pointer to the first element of the sequence, which is assigned to `foo` (a pointer). Therefore, `foo` now points to a valid block of memory with space for five elements of type `int`.
>
> Here, `foo` is a pointer, and thus, the first element pointed to by `foo` can be accessed either with the expression `foo[0]` or the expression `*foo` (both are equivalent). The second element can be accessed either with `foo[1]` or `*(foo+1)`, and so on...

由于使用动态内存分配机制，因此 `number_of_elements` 可以是一个变量，变量值在运行时才决定，例如：`p = new int[i];`。

---

声明普通数组与使用 `new` 分配动态内存的区别：

> There is a substantial difference between declaring a normal array and allocating dynamic memory for a block of memory using `new`. The most important difference is that the size of a regular array needs to be a *constant expression*, and thus its size has to be determined at the moment of designing the program, before it is run, whereas the dynamic memory allocation performed by `new` allows to assign memory during runtime using any variable value as size.

---

C++ 提供了两种标准机制来检查堆内存分配是否成功：

> The dynamic memory requested by our program is allocated by the system from the memory heap. However, computer memory is a limited resource, and it can be exhausted. Therefore, there are no guarantees that all requests to allocate memory using operator `new` are going to be granted by the system.
>
> C++ provides two standard mechanisms to check if the allocation was successful:

机制一：异常机制

> One is by handling exceptions. Using this method, an exception of type `bad_alloc` is thrown when the allocation fails. If this exception is thrown and it is not handled by a specific handler, the program execution is terminated.
>
> ```C++
> foo = new int [5];  // if allocation fails, an exception is thrown
> ```

机制二：返回空指针

> The other method is known as `nothrow`, and what happens when it is used is that when a memory allocation fails, instead of throwing a `bad_alloc` exception or terminating the program, the pointer returned by `new` is a *null pointer*, and the program continues its execution normally.
>
> This method can be specified by using a special object called `nothrow`, declared in header [`<new>`](http://www.cplusplus.com/%3Cnew%3E), as argument for `new`:
>
> ```C++
> foo = new (nothrow) int [5];
> ```
>
> In this case, if the allocation of this block of memory fails, the failure can be detected by checking if `foo` is a null pointer:
>
> ```C++
> int * foo;
> foo = new (nothrow) int [5];
> if (foo == nullptr) {
>   // error assigning memory. Take measures.
> }
> ```
>
> This `nothrow` method is likely to produce less efficient code than exceptions, since it implies explicitly checking the pointer value returned after each and every allocation. Therefore, the exception mechanism is generally preferred, at least for critical allocations. But `nothrow` mechanism is more simplicity.
>
> It is considered good practice for programs to always be able to handle failures to allocate memory, either by checking the pointer value (if `nothrow`) or by catching the proper exception.

## 动态内存回收

如果您不再需要动态分配的内存空间，可以使用 `delete` 运算符，删除之前由 `new` 运算符分配的内存，以便该内存可再次用于其它动态内存分配。语法如下：

```C++
// releases the memory of a single element allocated using new
delete bar;

// releases the memory allocated for arrays of elements using new and a size in brackets ([])
delete [] foo; // 不管所删除数组的维数多少，指针名前只用一对方括号 []
```

## Dynamic memory in C

> C++ integrates the operators `new` and `delete` for allocating dynamic memory. But these were not available in the C language; instead, it used a library solution, with the functions `malloc`, `calloc`, `realloc` and `free`, defined in the header [`<cstdlib>`](http://www.cplusplus.com/%3Ccstdlib%3E) (known as `<stdlib.h>` in C). The functions are also available in C++ and can also be used to allocate and deallocate dynamic memory.
>
> Note, though, that the memory blocks allocated by these functions are not necessarily compatible with those returned by `new`, so they should not be mixed; each one should be handled with its own set of functions or operators.

# 参考

http://www.cplusplus.com/doc/tutorial/pointers/

http://www.cplusplus.com/doc/tutorial/dynamic/

[Difference between pointer and reference in C ?](https://stackoverflow.com/questions/4995899/difference-between-pointer-and-reference-in-c)

[C++ 中的参数传递方式：传值、传地址、传引用总结](https://cdlwhm1217096231.github.io/C/C-%E4%B8%AD%E7%9A%84%E5%8F%82%E6%95%B0%E4%BC%A0%E9%80%92%E6%96%B9%E5%BC%8F%EF%BC%9A%E4%BC%A0%E5%80%BC%E3%80%81%E4%BC%A0%E5%9C%B0%E5%9D%80%E3%80%81%E4%BC%A0%E5%BC%95%E7%94%A8%E6%80%BB%E7%BB%93/)

[32/64 位系统支持多大内存？](https://www.crucial.cn/learn-with-crucial/memory/how-much-memory-does-your-windows-support)

