---
title: C/C++ 语言系列（六）复合数据类型之字符串
date: 2021-02-12 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

重点：区分下述四种形式：

```C
// 字符数组
char str[] = {'h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd', '\0'};
char str[] = "hello world";
// 字符串指针
char *str = "hello world";
// 字符串类
string str = "hello world";
```

# 字符数组

字符数组（Character sequences）：http://www.cplusplus.com/doc/tutorial/ntcs/

字符串实际上是使用 [null 字符](https://baike.baidu.com/item/Null/19660386) `\0` 终止的一维字符数组，如下：

![C 字符串](/img/cpp/c-strings.png)

代码如下：

```C
  // 字符串实际上是使用 null 字符 \0 终止的一维字符数组
  char str[] = {'h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd', '\0'};
  // 依据数组初始化规则，简化如下：
  char str2[] = "hello world";
  // conversion from string literal to 'char *' is deprecated
  char *str3 = "hello world";

  // str: hello world size: 12
  cout << "str: "  << str << " size: " << sizeof(str) << endl;
  // str2: hello world size: 12
  cout << "str2: " << str2 << " size: " << sizeof(str2) << endl;
  // 'sizeof (str3)' will return the size of the pointer, not the array itself
  // str3: hello world size: 8
  cout << "str3: " << str3 << " size: " << sizeof(str3) << endl;
```

`cstring` 头文件提供了大量的函数，用来操作 *C strings* and arrays，参考：http://www.cplusplus.com/reference/cstring/

# 字符串指针

用字符数组和字符串指针都可实现字符串的存储和运算，但是两者是有区别的：

* 字符数组是一个数组，每个元素的值都可以改变。
* 而字符串指针指向的是一个**常量字符串**，它被存放在程序的**静态数据区，一旦定义就不能改变**。

这是最重要的区别。下面的代码在运行期间将会出错：

```C
  str2[1] = 'a';      // hallo world
  *(str3 + 1) = 'a';  // 运行时出错。因为不能改变字符串常量的值
```

# 字符串类

`string` 头文件提供了 `string` 类，参考：http://www.cplusplus.com/reference/string/

# 参考

https://www.runoob.com/cplusplus/cpp-strings.html

[C++输出char型变量与字符串的地址](https://blog.csdn.net/u014082714/article/details/45498527)

https://stackoverflow.com/questions/1524356/c-deprecated-conversion-from-string-constant-to-char