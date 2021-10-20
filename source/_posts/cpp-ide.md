---
title: C/C++ 语言系列（一）IDE 准备
date: 2021-02-01 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# 环境准备

打开 VScode，进入 `Extensions`，二选一搜索并安装以下扩展：

## 方式一：官方插件 C/C++

[Using Clang in Visual Studio Code](https://code.visualstudio.com/docs/cpp/config-clang-mac) 教程指引如何使用官方插件 [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) 进行构建与调试。

按教程配置 `.vscode` 文件夹的三个文件：

- `tasks.json` （编译器构建设置）
- `launch.json` （调试器设置）
- `c_cpp_properties.json` （编译器路径和 IntelliSense 设置）

`⇧⌘B` 编译源文件：

```bash
/usr/bin/clang++ -std=c++17 -stdlib=libc++ -g /Users/wuqd/Documents/workspace/cpp/HelloWorld.cpp -o /Users/wuqd/Documents/workspace/cpp/HelloWorld.out
```

编译成功后：

* 可以在「终端」输入 `./HelloWorld.out` 以直接运行编译文件；

* 还可以按 `F5` 调试。

## 方式二：三方插件 Code Runner

安装第三方插件：[Code Runner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner)

`settings.json` 新增配置如下，将编译文件的后缀名改为 `.out`，以便 .gitignore：

```json
    "code-runner.executorMap": {
      "cpp": "cd $dir && g++ $fileName -o $fileNameWithoutExt.out && $dir$fileNameWithoutExt.out"
    }
```

`⌃⌥N` 编译并运行：

```bash
cd "/Users/wuqd/Documents/workspace/cpp/" && g++ HelloWorld.cpp -o HelloWorld.out && "/Users/wuqd/Documents/workspace/cpp/"HelloWorld.out
```

 # 常见问题

## DIFFERENCE BETWEEN g++ & gcc

> [GCC](https://www.geeksforgeeks.org/builtin-functions-gcc-compiler/) 代表 [GNU Compiler Collections](https://www.geeksforgeeks.org/gcc-command-in-linux-with-examples/)，主要用于编译 C 和 C++ 语言。它还可以用于编译 Objective C 和 Objective C++。编译源代码文件时需要的最重要的选项是源程序的名称，其余每个参数都是可选的，如警告、调试、链接库、目标文件等。
>
> [g++ 命令](https://www.geeksforgeeks.org/compiling-with-g-plus-plus/)是 GNU C++ 编译器调用的命令，用于源代码的预处理、编译、汇编和链接以生成可执行文件。
>
> 
>
> | g++                                                          | gcc                                                          |
> | :----------------------------------------------------------- | :----------------------------------------------------------- |
> | g++ is used to compile C++ program.                          | gcc is used to compile C program.                            |
> | g++ can compile any `.c` or `.cpp` files but they will be treated as C++ files only. | gcc can compile any `.c` or `.cpp` files but they will be treated as C and C++ respectively. |
> | Command to compile C++ program through g++ is `g++ fileName.cpp -o binary` | command to compile C program through gcc is `gcc fileName.c -o binary` |
> | Using g++ to link the object files, files automatically links in the std C++ libraries. | gcc does not do this.                                        |
> | g++ compiles with more predefined macros.                    | gcc compiles C++ files with more number of predefined macros. Some of them are #define __GXX_WEAK__ 1, #define __cplusplus 1, #define __DEPRECATED 1, etc |

## DIFFERENCE BETWEEN gcc & make

> gcc是编译器 而make不是 make是依赖于Makefile来编译多个源文件的工具 在Makefile里同样是用gcc(或者别的编译器)来编译程序.
>
> gcc是编译一个文件，make是编译多个源文件的工程文件的工具。
>
> make是一个命令工具，是一个解释makefile中指令的命令工具。
>
> make就是一个gcc/g++的调度器，通过读入一个文件（默认文件名为Makefile或者makefile），执行一组以gcc/g++为主的shell命令序列。输入文件主要用来记录文件之间的依赖关系和命令执行顺序。
>
> gcc是编译工具；
>
> make是定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译；
>
> 也就是说make是调用gcc的。

## cannot edit in read-only editor

![](/img/cpp/auto_guess_encoding.png)

## GBK to UTF-8 乱码问题

https://www.cnblogs.com/kingsonfu/p/11010086.html

![](/img/cpp/cannot_edit_in_read-only_editor.png)

# 参考

[Using Clang in Visual Studio Code](https://code.visualstudio.com/docs/cpp/config-clang-mac)

http://www.cplusplus.com/doc/tutorial/introduction/

[C++ 在线编译器](http://www.dooccn.com/cpp/)

[C++ Insights](https://cppinsights.io/)

[C++ Compiler Explorer](https://gcc.godbolt.org/)

VSCode 其它插件推荐：

* C/C++ Clang Command Adapter
* C++ Intellisense