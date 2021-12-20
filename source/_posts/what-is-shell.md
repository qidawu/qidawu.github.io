---
title: GNU/Bash 系列（一）什么是 Shell？
date: 2015-03-01 20:09:48
updated:
tags: GNU/Linux
---

![What's a shell](https://wizardzines.com/comics/shell/shell.png)

# 什么是 Shell ？

* 一个命令行解释器，用于将用户输入的命令转换为系统操作；
* Shell 既是交互式命令语言（CMD），也是脚本编程语言（Script）；
* Shell 有很多内置在其源代码中的命令。这些命令是内置的，所以 Shell 不必到磁盘上搜索它们，执行速度因此加快。不同的 Shell 内置命令有所不同；

# 有哪些 Shell ？

## GUI (Graphical User Interface)

* GNOME
* KDE
* ……

## CLI (Command Line Interface)

CLI 实际上是一个程序，比如 /bin/bash 。它是一个实实在在的程序，它打印提示符(PROMPT)，接受用户输入的命令，分析命令序列并执行然后返回结果。

* Bourne Shell (sh)
  * 即 sh，是影响最广的 shell 。 1977 年由 Stephen Bourne 在贝尔实验室编写。虽无明文规定，但已成为事实上的标准；
  * 引入了 shell 通用的、基础的功能，例如管道、变量、条件判断、循环等；
  * 全部类 Unix 系统，都至少有一个与 sh 兼容的shell；
  * sh 一般位于 /bin/sh。目前大部分系统，/bin/sh 都是一个链接，指向一个兼容 sh 的、功能更丰富的 shell，如 bash；
* C Shell (csh)
  * 即 csh，1978年由 Bill Joy 在伯克利大学毕业后编写；
  * Bill Joy，Sun 联合创始人，vi 作者，BSD 作者；
  * csh 更加注重交互式使用而非脚本应用，引入了历史功能、别名、目录栈、作业控制等功能；
* Bourne Again Shell (bash)
  * 即 bash，1989年由 Brian Fox 为 GNU 项目编写。
  * 综合了 sh、csh、ksh 等各种 shell 的特性；
  * 名字有双关含义，既是字面上的意思，用于替换 sh，也隐含为了 GNU 而 born again 的意思；
  * 是 Mac OS X 和大部分 Linux 发行版的默认 shell；
* Korn Shell (ksh)
* Z Shell (zsh)
* TENEX C Shell (tcsh)
* ……

# GUI 和 CLI 如何切换？

切换到 tty1 ~ tty6 终端：`Ctrl + Alt + [F1 ~ F6]`

切换到 GUI：`Ctrl + Alt + [F7]`