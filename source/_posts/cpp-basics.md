---
title: C/C++ 语言系列（二）基础语法入门
date: 2021-02-02 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# 一、预处理器

| 指令                                       | 描述                                                         |
| :----------------------------------------- | :----------------------------------------------------------- |
| `#include`                                 | 包含一个源代码文件（[扩展名为 `.h` 的头文件](https://www.runoob.com/cprogramming/c-header-files.html)） |
| `#define`<br/>`#undef`                     | 定义宏<br/>取消已定义的宏                                    |
| `#if`<br/>`#else`<br/>`#elif`<br/>`#endif` | 条件编译                                                     |

参考：

* [C 预处理器](https://www.runoob.com/cprogramming/c-preprocessors.html)
* [Preprocessor directives](http://www.cplusplus.com/doc/tutorial/preprocessor/)

# 二、基础语法

## 1、标识符（identifier）

是用来标识**变量、函数、类、模块**，或任何其他用户**自定义项目**的名称。

一个标识符只能以：

* 字母 `A-Z` 或 `a-z` 或下划线 `_` 开始；
* 后跟零个或多个字母、下划线和数字（`0-9`）。

## 2、保留字（关键字）

## 3、注释

单行注释（行注释）：`//`

多行注释（块注释）：`/* ... */`

## 4、变量与常量

[C++ 变量类型](https://www.runoob.com/cplusplus/cpp-variable-types.html)

[C++ 数据类型](https://www.runoob.com/cplusplus/cpp-data-types.html)

[C++ 修饰符类型](https://www.runoob.com/cplusplus/cpp-modifier-types.html)

[C++ 变量作用域](https://www.runoob.com/cplusplus/cpp-variable-scope.html)

### 4.1、基本数据类型

C/C++ 基本数据类型：

* 一种**布尔类型**
* 一种**字符类型**
* 一种**整数型**
* 两种**浮点型**

| 类型     | 关键字   | 存储大小                    | 备注                       |
| :------- | :------- | --------------------------- | -------------------------- |
| 布尔型   | `bool`   | `sizeof(bool)` = `1` 字节   |                            |
| 字符型   | `char`   | `sizeof(char)` = `1` 字节   |                            |
| 整型     | `int`    | `sizeof(int)` = `4` 字节    |                            |
| 浮点型   | `float`  | `sizeof(float)` = `4` 字节  | C++ 中，小数默认为浮点型。 |
| 双浮点型 | `double` | `sizeof(double)` = `8` 字节 |                            |

![基本数据类型](/img/cpp/data_type.png)

### 4.2、修饰符

C++ 允许在 `char`、`int` 和 `double` 数据类型前放置修饰符。修饰符用于改变基本类型的含义，所以它更能满足各种情境的需求。

| 修饰符     | `int` | `double` | `char` |
| ---------- | ----- | -------- | ------ |
| `signed`   | Y     |          | Y      |
| `unsigned` | Y     |          | Y      |
| `long`     | Y     | Y        |        |
| `short`    | Y     |          |        |

### 4.3、作用域

 * 作用域可分为：
    * 全局作用域
    * 局部作用域
    * 语句作用域
 * 作用域优先级：范围越小，优先级越高
 * 如果希望在局部作用域中使用同名的全局变量，可以在该变量前使用：**作用域运算符** `::`

```C++
#include <iostream>
using namespace std;

// 全局变量
int x = 10;

int main()
{
  // 局部变量
  int x = 100;
  cout << x << endl; // 100
  cout << ::x << endl; // 10
}
```

### 4.4、常量定义

在 C++ 中，有两种简单的定义 [C++ 常量](https://www.runoob.com/cplusplus/cpp-constants-literals.html)的方式：

- 使用 `#define` 预处理器进行**宏定义**。
- 使用 `const` 关键字。

```C++
#include <iostream>
using namespace std;

// 定义常量（使用预处理器）
#define LENGTH 10   
#define WIDTH  5
#define NEWLINE '\n'
 
int main()
{
  // 定义常量（使用 const 关键字）
  const int  LENGTH = 10;
  const int  WIDTH  = 5;
  const char NEWLINE = '\n';
  
  return 0;
}
```

## 5、运算符

### 5.1、位运算符

### 5.2、算术运算符

| 运算符 | 描述                                                         | 实例             |
| :----- | :----------------------------------------------------------- | :--------------- |
| `+`    | 把两个操作数相加                                             | A + B 将得到 30  |
| `-`    | 从第一个操作数中减去第二个操作数                             | A - B 将得到 -10 |
| `*`    | 把两个操作数相乘                                             | A * B 将得到 200 |
| `/`    | 分子除以分母                                                 | B / A 将得到 2   |
| `%`    | 取模运算符，整除后的余数                                     | B % A 将得到 0   |
| `++`   | [自增运算符](https://www.runoob.com/cplusplus/cpp-increment-decrement-operators.html)，整数值增加 1 | A++ 将得到 11    |
| `--`   | [自减运算符](https://www.runoob.com/cplusplus/cpp-increment-decrement-operators.html)，整数值减少 1 | A-- 将得到 9     |

### 5.3、赋值运算符

| 运算符 | 描述                                                         | 实例                            |
| :----- | :----------------------------------------------------------- | :------------------------------ |
| `=`    | 简单的赋值运算符，把右边操作数的值赋给左边操作数             | C = A + B 将把 A + B 的值赋给 C |
| `+=`   | 加且赋值运算符，把右边操作数加上左边操作数的结果赋值给左边操作数 | C += A 相当于 C = C + A         |
| `-=`   | 减且赋值运算符，把左边操作数减去右边操作数的结果赋值给左边操作数 | C -= A 相当于 C = C - A         |
| `*=`   | 乘且赋值运算符，把右边操作数乘以左边操作数的结果赋值给左边操作数 | C *= A 相当于 C = C * A         |
| `/=`   | 除且赋值运算符，把左边操作数除以右边操作数的结果赋值给左边操作数 | C /= A 相当于 C = C / A         |
| `%=`   | 求模且赋值运算符，求两个操作数的模赋值给左边操作数           | C %= A 相当于 C = C % A         |
| `<<=`  | 左移且赋值运算符                                             | C <<= 2 等同于 C = C << 2       |
| `>>=`  | 右移且赋值运算符                                             | C >>= 2 等同于 C = C >> 2       |
| `&=`   | 按位与且赋值运算符                                           | C &= 2 等同于 C = C & 2         |
| `^=`   | 按位异或且赋值运算符                                         | C ^= 2 等同于 C = C ^ 2         |
| `\|=`  | 按位或且赋值运算符                                           | C \|= 2 等同于 C = C \| 2       |

### 5.4、关系运算符

| 运算符 | 描述                                                         | 实例              |
| :----- | :----------------------------------------------------------- | :---------------- |
| `==`   | 检查两个操作数的值是否相等，如果相等则条件为真。             | (A == B) 不为真。 |
| `!=`   | 检查两个操作数的值是否相等，如果不相等则条件为真。           | (A != B) 为真。   |
| `>`    | 检查左操作数的值是否大于右操作数的值，如果是则条件为真。     | (A > B) 不为真。  |
| `<`    | 检查左操作数的值是否小于右操作数的值，如果是则条件为真。     | (A < B) 为真。    |
| `>=`   | 检查左操作数的值是否大于或等于右操作数的值，如果是则条件为真。 | (A >= B) 不为真。 |
| `<=`   | 检查左操作数的值是否小于或等于右操作数的值，如果是则条件为真。 | (A <= B) 为真。   |

### 5.5、逻辑运算符

| 运算符 | 描述                                                         | 实例                 |
| :----- | :----------------------------------------------------------- | :------------------- |
| `&&`   | 称为逻辑与运算符。如果两个操作数都 true，则条件为 true。     | (A && B) 为 false。  |
| `\|\|` | 称为逻辑或运算符。如果两个操作数中有任意一个 true，则条件为 true。 | (A \|\| B) 为 true。 |
| `!`    | 称为逻辑非运算符。用来逆转操作数的逻辑状态，如果条件为 true 则逻辑非运算符将使其为 false。 | !(A && B) 为 true。  |

### 5.6、其它运算符

| 运算符                   | 描述                                                         |
| :----------------------- | :----------------------------------------------------------- |
| `::`                     | 作用域运算符 Scope operator，用于引用全局变量 `::code`、引用某个命名空间的函数或变量 `namespace::code` 等等。 |
| `&`                      | [取地址运算符 Address-of operator (&)](https://www.runoob.com/cplusplus/cpp-pointer-operators.html) 返回变量的地址。例如 `&a` 将给出变量的实际内存地址。 |
| `*`                      | [间接寻址运算符 Dereference operator (*)](https://www.runoob.com/cplusplus/cpp-pointer-operators.html) 指向一个变量。例如，`*var` 返回操作数所指定地址的变量的值。 |
| `.`（点）和 `->`（箭头） | [成员运算符](https://www.runoob.com/cplusplus/cpp-member-operators.html)用于引用**类**、**结构体**和**共用体**的成员。 |
| `sizeof`                 | [sizeof 运算符](https://www.runoob.com/cplusplus/cpp-sizeof-operator.html)返回变量的存储大小。例如，`sizeof(int)` 返回 `4` 个字节。 |
| `Cast`                   | [强制转换运算符](https://www.runoob.com/cplusplus/cpp-casting-operators.html)把一种数据类型转换为另一种数据类型。例如，`int(2.2000)` 将返回 2。 |
| `new`                    | 在堆上动态内存分配。参考：[Dynamic memory](http://www.cplusplus.com/doc/tutorial/dynamic/) |
| `delete`                 | 在堆上进行内存回收。                                         |

提取运算符 `>>`

插入运算符 `<<`

## 6、控制语句

C 语言有九种控制语句。 可分成以下三类：

### 6.1、选择语句

- `if`、`else` 语句
- `switch` 语句

### 6.2、循环语句

- `while` 语句
- `do while` 语句
- `for` 语句

### 6.3、跳转语句

- `break` 语句
- `continue` 语句
- `goto`语句（此语句尽量少用，因为这不利**结构化程序设计**，滥用它会使程序流程无规律、可读性差）

# 参考

https://www.cplusplus.com/doc/tutorial/

https://www.cplusplus.com/doc/tutorial/variables/

https://www.cplusplus.com/doc/tutorial/namespaces/