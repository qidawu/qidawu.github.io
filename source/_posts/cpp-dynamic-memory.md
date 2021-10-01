---
title: C/C++ 语言系列（十二）动态内存分配与回收
date: 2021-03-01 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# C++ 语言

## 动态内存分配 new

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
> // error assigning memory. Take measures.
> }
> ```
>
> This `nothrow` method is likely to produce less efficient code than exceptions, since it implies explicitly checking the pointer value returned after each and every allocation. Therefore, the exception mechanism is generally preferred, at least for critical allocations. But `nothrow` mechanism is more simplicity.
>
> It is considered good practice for programs to always be able to handle failures to allocate memory, either by checking the pointer value (if `nothrow`) or by catching the proper exception.

## 动态内存回收 delete

如果您不再需要动态分配的内存空间，可以使用 `delete` 运算符，删除之前由 `new` 运算符分配的内存，以便该内存可再次用于其它动态内存分配。语法如下：

```C++
// releases the memory of a single element allocated using new
delete bar;

// releases the memory allocated for arrays of elements using new and a size in brackets ([])
delete [] foo; // 不管所删除数组的维数多少，指针名前只用一对方括号 []
```

# C 语言

> C++ integrates the operators `new` and `delete` for allocating dynamic memory. But these were not available in the C language; instead, it used a library solution, with the functions `malloc`, `calloc`, `realloc` and `free`, defined in the header [`<cstdlib>`](http://www.cplusplus.com/%3Ccstdlib%3E) (known as `<stdlib.h>` in C). The functions are also available in C++ and can also be used to allocate and deallocate dynamic memory.
>
> Note, though, that the memory blocks allocated by these functions are not necessarily compatible with those returned by `new`, so they should not be mixed; each one should be handled with its own set of functions or operators.

## 动态内存分配 malloc, calloc, realloc

## 动态内存回收 free

# 参考

http://www.cplusplus.com/doc/tutorial/dynamic/