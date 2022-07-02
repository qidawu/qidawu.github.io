---
title: C/C++ 语言系列（五）复合数据类型之数组
date: 2021-02-10 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# 一维数组

一维数组声明：`type arrayName[ arraySize ];`

```C++
  // 声明数组
  int array1[5];
```

一维数组初始化：

```C++
  // 初始化数组
  float array1[5] = {1, 2, 3, 4, 5};
  // 初始化数组（省略数组大小声明，默认大小为初始化时元素的个数）
  float array2[] = {1, 2, 3, 4, 5};
```

一维数组访问：

```C++
  // 通过索引逐个访问数组元素，并赋值
  array1[0];  // 1
  array1[1];  // 2
  array1[2];  // 3
  array1[3];  // 4
  array1[4];  // 5
```

# 多维数组

多维数组声明：`type name[ size1 ][ size2 ]...[ sizeN ];`

```C++
  // 声明一个三维 5 . 10 . 4 整型数组
  int array1[5][10][4];
```

二维数组初始化：

```C++
  // 多维数组可以通过在括号内为每行指定值来进行初始化。下面是一个带有 3 行 4 列的数组。
  int a[3][4] = {  
   {0, 1, 2, 3} ,   /*  初始化索引号为 0 的行 */
   {4, 5, 6, 7} ,   /*  初始化索引号为 1 的行 */
   {8, 9, 10, 11}   /*  初始化索引号为 2 的行 */
  };

  // 内部嵌套的括号是可选的，下面的初始化与上面是等同的：
  int a[3][4] = {0,1,2,3,4,5,6,7,8,9,10,11};
```

二维数组访问：

```C++
  for (int i = 0; i < sizeof(a) / sizeof(a[0]); i++)
  {
    for (int j = 0; j < sizeof(a[0]) / sizeof(a[0][0]); j++)
    {
      cout << a[i][j] << endl;
    }
  }
```

二维数组大小及长度获取：

```C++
  // 数组总大小：48
  sizeof(a)
  // 数组第一维大小：16
  sizeof(a[0])
  // 数组第一维长度：3
  sizeof(a) / sizeof(a[0])
  // 数组第二维长度：4
  sizeof(a[0]) / sizeof(a[0][0])
```

# 数组名

**数组名**是指向数组中第一个元素的**常指针（常量指针）**，因此 `arrayName` 等价于 `&arrayName[0]`，验证如下：

```C++
  int arrayName[] = {0, 1, 2, 3, 4};
  cout << arrayName << endl;     // 0x7ffeec364620
  cout << &arrayName[0] << endl; // 0x7ffeec364620
```

由于数组是**常指针**，因此自身无法执行算术运算：

```C++
  // 表达式必须是可修改的左值
  // invalid operands to binary expression
  arrayName += 1;
  arrayName -= 1;
  arrayName++;
  arrayName--;
  ++arrayName;
  --arrayName;
```

# 数组构成

* 存储一个**固定大小**的**相同数据类型**元素的顺序集合。
* 由**连续的内存地址**组成。

验证如下：

```C++
  for (int i = 0; i < 5; i++) {
    cout << &arrayName[i] << endl;
  }
```

输出如下：

```
0x7ffeec364620
0x7ffeec364624
0x7ffeec364628
0x7ffeec36462c
0x7ffeec364630
```

分析上述输出结果，由于 1 个内存单元的大小是 `8 bits`，即一个字节。而一个 `int` 类型的变量占用 4 个字节，因此上述 16 进制表示的内存地址的递增步长为 4。

# 获取数组大小

通过 `sizeof` 运算符，确认该数组大小为 20 字节：

```C++
sizeof(arrayName) // 20
```

# 获取数组长度

`sizeof` 运算符用于获取变量的存储大小（即所占内存字节数），由于数组中每个元素的类型都是一样的、所占字节数亦然，因此可以计算如下：

```C++
sizeof(arrayName) / sizeof(arrayName[0]);  // 5
```

参考：[C 语言数组传入函数获取数组长度的方法](https://zhuanlan.zhihu.com/p/104531696VVV)

# 遍历数组元素

使用数组索引访问数组元素：

```C++
  for (int i = 0; i < 5; i++) {
    cout << arrayName[i] << endl;	// 1 2 3 4 5
  }
```

# 指针数组 VS 数组指针

![指针数据 VS 数组指针](/img/c/pointer_array.jpg)

## 指针数组

指针数组，表示“存储指针的数组”，即定义一个数组，其每个元素都是指针：

```C++
  int *p1[10];
```

## 数组指针

数组指针，表示“指向数组的指针”，即定义一个指针，指向数组：

```C++
  int *p2 = new int[10];
```

常用于定义函数的**形式参数**：

```C++
  void func(int *p);
```

优点：数组作为**常指针**不能执行算术运算，但**数组指针**可以。下面通过递增指针，以顺序访问数组元素：

```C++
  int arrayName[5] = {1, 2, 3, 4, 5};  // &arrayName = 0x7ffee7376620
  int *p = arrayName;

  for (int i = 0; i < 5; i++)
  {
    cout << p << endl;  // 0x7ffee7376620 0x7ffee7376624 0x7ffee7376628 0x7ffee737662c 0x7ffee7376630
    cout << *p++ << endl;  // 1 2 3 4 5
  }
```

# 传递数组给函数

## 三种方式

有三种方式传递数组给函数。

方式一：形式参数是一个**数组指针**（指向数组的指针）。

```C++
void func(int *p)
{
  cout << sizeof(p) << endl;  // 返回指针长度 8 bytes
}
```

方式二：形式参数是一个已定义大小的数组。

```C++
void func(int p[5])
{
  // sizeof on array function parameter will return size of 'int *' instead of 'int [5]'
  cout << sizeof(p) << endl;  // 返回指针长度 8 bytes
}
```

方式三：形式参数是一个未定义大小的数组。

```C++
void func(int p[])
{
  // sizeof on array function parameter will return size of 'int *' instead of 'int []'
  cout << sizeof(p) << endl;  // 返回指针长度 8 bytes
}
```

## 注意点

无论使用何种方式，**函数内都无法获取数组长度**，需要使用单独变量将数组长度作为参数传入。

```C++
void func(int p[], int length)
{
  for (int i = 0; i < 5; i++)
  {
    cout << p[i] << endl;  // 1 2 3 4 5
  }
}
```

# 参考

http://www.cplusplus.com/doc/tutorial/arrays/

https://www.runoob.com/cplusplus/cpp-arrays.html

