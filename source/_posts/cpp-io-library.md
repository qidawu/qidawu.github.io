---
title: C/C++ 语言系列（十一）I/O 库总结
date: 2021-02-26 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# Input/Output library

下图是 C++ 提供的输入/输出库，其中：

* `<xxx>` 表示：头文件
* 白框表示：类（Classes）
* 黑框表示：对象（Objects）

![Input/Output library](/img/cpp/iostream.gif)

> The iostream library is an object-oriented library that provides input and output functionality using streams.



> A stream is an abstraction that represents a device on which input and ouput operations are performed. A stream can basically be represented as a source or destination of characters of indefinite length.



> Streams are generally associated to a physical source or destination of characters, like a disk file, the keyboard, or the console, so the characters gotten or written to/from our abstraction called stream are physically input/output to the physical device. For example, file streams are C++ objects to manipulate and interact with files; Once a file stream is used to open a file, any input or output operation performed on that stream is physically reflected in the file.

To operate with streams, C++ provides the standard `iostream` library, which contains the following elements:

![Elements of the iostream library](/img/cpp/elements_of_the_iostream_library.png)

## Basic class templates

> The base of the iostream library is the hierarchy of class templates. The class templates provide most of the functionality of the library in a type-independent fashion.

## Class template instantiations

> The library incorporates two standard sets of instantiations of the entire iostream class template hierarchy: 
>
> * The narrow-oriented (`char` type) instantiation is probably the better known part of the iostream library. Classes like `ios`, `istream` and `ofstream` are narrow-oriented. The diagram on top of this page shows the names and relationships of narrow-oriented classes.
>
> * The classes of the wide-oriented (`wchar_t`) instatiation follow the same naming conventions as the narrow-oriented instantiation but with the name of each class and object prefixed with a `w` character, forming `wios`, `wistream` and `wofstream`, as an example.

## Objects

> As part of the iostream library, the header file `<iostream>` declares certain objects that are used to perform input and output operations on the standard input and output.
>
> They are divided in two sets: 
>
> * narrow-oriented objects: `cin`, `cout`, `cerr` and `clog` 
> * wide-oriented objects: `wcin`, `wcout`, `wcerr` and `wclog`

## Manipulators

> Manipulators are global functions designed to be used together with insertion (`<<`) and extraction (`>>`) operators performed on `iostream` stream objects. They generally modify properties and formatting settings of the streams.

# 常用头文件

下列这些头文件在 C++ 编程中很常用，下面分别介绍：

| 头文件       | 函数和描述                                                   |
| :----------- | :----------------------------------------------------------- |
| `<iostream>` | 该文件定义了 `cin`、`cout`、`cerr` 和 `clog` 对象，用于输入输出。 |
| `<iomanip>`  | 该文件通过所谓的参数化的流操纵器（比如 `setw`、`setfill`、`setprecision`），来声明对执行标准化 I/O 有用的服务。 |
| `<fstream>`  | 该文件定义了 `ifstream`、`ofstream`、`fstream` 对象，用于文件读写。 |

## &lt;iostream&gt;

下图摘录了 `iostream` 类的继承关系及成员函数，如下：

![iostream classes](/img/cpp/iostream_class.png)

http://www.cplusplus.com/doc/tutorial/basic_io/

http://www.cplusplus.com/reference/iolibrary/

### output stream

输出流与流插入运算符 `<<` 配合使用。`<iostream>` 提供了下列三种输出流对象：

- `cout` 标准输出流（默认设备是显示器屏幕）
- `cerr` 无缓冲标准错误输出流（默认设备是显示器屏幕）
- `clog` 有缓冲标准错误输出流（默认设备是打印机）

此外，`<iostream>` 还提供了一个常用的操纵符：

- `endl` Insert newline and flush

例子：

```C++
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

### input stream

输入流与流提取运算符 `>>` 配合使用。`<iostream>` 提供了下列一种输入流对象：

- `cin` 标准输入流（默认设备是键盘）

例子：

```C++
cin >> name;
cin >> age;

// 流提取运算符 >> 在一个语句中可以多次使用，如果要求输入多个数据，可以使用如下语句:
cin >> name >> age;
```

## &lt;iomanip&gt; 流操纵器

常用流操纵器函数如下：

| 函数                                                         | 作用                  |
| ------------------------------------------------------------ | --------------------- |
| [`setw`](http://www.cplusplus.com/reference/iomanip/setw/)   | Set field width       |
| [`setfill`](http://www.cplusplus.com/reference/iomanip/setfill/) | Set fill character    |
| [`setprecision`](http://www.cplusplus.com/reference/iomanip/setprecision/) | Set decimal precision |
| ...                                                          |                       |

`setw` 函数用于设置字段的宽度，只对紧接着的输出产生作用。当后面紧跟着的输出字段长度小于 `n` 的时候，在该字段前面默认用**空格补齐**，当输出字段长度大于 `n` 时，全部整体输出。如下：

![setw](/img/cpp/cpp-setw.svg)

例子：

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

参考：

https://www.runoob.com/w3cnote/cpp-func-setw.html

## &lt;fstream&gt; 文件读写

`<fstream>` 定义了下面三个对象，用于文件读写：

| 数据类型   | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| `ofstream` | 该数据类型表示输出文件流，用于创建文件并向文件写入信息。     |
| `ifstream` | 该数据类型表示输入文件流，用于从文件读取信息。               |
| `fstream`  | 该数据类型通常表示文件流，且同时具有 ofstream 和 ifstream 两种功能，这意味着它可以创建文件，向文件写入信息，从文件读取信息。 |

例子：

```C++
#include <iostream>
#include <fstream>
using namespace std;

void output() {
  ofstream myfile;
  // 打开文件
  myfile.open("/Users/wuqd/Desktop/test", ios::app);  // ios::app 表示追加模式。所有写入都追加到文件末尾。
  // 写入文件
  myfile << "helloworld" << endl;
  // 关闭文件
  myfile.close();
}

void input() {
  ifstream in;
  // 打开文件
  in.open("/Users/wuqd/Desktop/test");
  char data[100];
  // 读取文件
  while (in >> data) {
    // 输出到屏幕
    cout << data << endl;
  }
  // 关闭文件
  in.close();
}

int main() {
  output();
  input();
  return 0;
}
```

参考：

https://www.cplusplus.com/doc/oldtutorial/files/

https://www.runoob.com/cplusplus/cpp-files-streams.html



