---
title: C/C++ 语言系列（十）I/O 库总结
date: 2021-02-26 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

https://www.runoob.com/cplusplus/cpp-files-streams.html

# 1、基本概念

I/O 设备：

* 输入设备：计算机用于接受外界信息的设备，如键盘、鼠标、扫描仪、USB 设备等。
* 输出设备：将计算机处理完毕的数据送往外界的设备，如显示器、打印机、绘图仪等。

早期计算机系统输入/输出的实现：

* 设置有专门的输入/输出操作指令、利用它们来完成 I/O 任务。
* 规范化：把 I/O 按**文件方式**进行管理，通过函数或流类进行实现。
* 优点：把跟硬件相关的部分集中到有限的几个函数或流上，提高了程序的可移植性，使用方面有较好的规范，方便使用。

文件的基本概念：

* 文件：计算机系统中能够以一个唯一的名字为标识进行读写访问的信息的集合。它用于存储程序和数据。采用文件还可用于提高程序和设备的无关性。
* 设备无关性：是指程序不仅
* 构造式文件：
* 流式文件：

C / C++ 语言本身都不包含直接操纵底层硬件设备的输入/输出语句，输入/输出是通过函数库与类库来完成的。

# 2、处理流程

# 3、Input/Output library

![Input/Output library](/img/cpp/iostream.gif)

http://www.cplusplus.com/reference/iolibrary/

- `cin` 标准输入（默认设备是键盘）
- `cout` 标准输出（默认设备是显示器屏幕）
- `cerr` 无缓冲标准错误输出（默认设备是显示器屏幕）
- `clog` 有缓冲标准错误输出（默认设备是打印机）
- `endl` 强制输出 buffer。

| 运算符 | 描述         |
| ------ | ------------ |
| `<<`   | 流插入运算符 |
| `>>`   | 流提取运算符 |

`strlen` 字符串长度

`strcmp` 字符串比较

`strcpy` 字符串拷贝

# 4、DEMO

## DEMO 1

```C
// 定义一些头文件，这些头文件包含了程序中必需的或有用的信息。
#include <iostream>
// 告诉编译器使用 std 命名空间。命名空间是 C++ 中一个相对新的概念。使用该命名空间之后，std::cout 可以简写为：cout
using namespace std;

// 主函数，程序从这里开始执行。
int main()
{
  // 在屏幕上输出 "Hello World"
  cout << "hello world" << std::endl;
  
  // 终止 main() 函数，并向调用进程返回值 0。
  return 0;
}

// 使用 g++ 编译器，编译 cpp 源文件为可执行文件，并执行之
// cd "/Users/wuqd/Documents/workspace/cpp/" && g++ HelloWorld.cpp -o HelloWorld && "/Users/wuqd/Documents/workspace/cpp/"HelloWorld
```

运行结果：`hello world`

## DEMO 2

```C
#include <iostream>
#include <iomanip>
using namespace std;
 
/* 
 * 测试 I/O
 */
int main()
{
  double pi = 3.1415926;

  // 默认右补位，可以使用 setf() 进行左补位
  cout.setf(ios::left);

  // setprecision() 包含小数点
  cout << setfill('*') << setw(5) << setprecision(3) << pi << endl;

  return 0;
}
```

运行结果：`3.14*`

# 参考

http://www.cplusplus.com/doc/tutorial/basic_io/