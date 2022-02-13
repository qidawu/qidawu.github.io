---
title: GNU/Bash 系列（一）什么是 Shell？
date: 2015-03-01 20:09:48
updated:
tags: GNU/Linux
---

# 什么是 Shell ？

参考维基百科：https://en.wikipedia.org/wiki/Shell_(computing)

> In [computing](https://en.wikipedia.org/wiki/Computing), a **shell** is a computer program which exposes an [operating system](https://en.wikipedia.org/wiki/Operating_system)'s services to a human user or other programs. In general, operating system shells use either a [command-line interface](https://en.wikipedia.org/wiki/Command-line_interface) (CLI) or [graphical user interface](https://en.wikipedia.org/wiki/Graphical_user_interface) (GUI), depending on a computer's role and particular operation. It is named a shell because it is the outermost layer around the operating system.

![What's a shell](https://wizardzines.com/comics/shell/shell.png)

* 一个命令行解释器，用于将用户输入的命令转换为系统操作；
* Shell 既是交互式命令语言（CMD），也是脚本编程语言（Script）；
* Shell 有很多内置在其源代码中的命令。这些命令是内置的，所以 Shell 不必到磁盘上搜索它们，执行速度因此加快。不同的 Shell 内置命令有所不同；

# 有哪些 Shell ？

## GUI (Graphical User Interface)

* GNOME
* KDE
* ……

## CLI (Command Line Interface)

* Bourne Shell (sh)
* Bourne Again Shell (Bash)
* C Shell (csh)
  * 即 csh，1978年由 Bill Joy 在伯克利大学毕业后编写；
  * Bill Joy，Sun 联合创始人，vi 作者，BSD 作者；
  * csh 更加注重交互式使用而非脚本应用，引入了历史功能、别名、目录栈、作业控制等功能；
* TENEX C Shell (tcsh)
* Korn Shell (ksh)
* Z Shell (Zsh)
* ……

### sh

https://en.wikipedia.org/wiki/Bourne_shell

* 即 sh，是影响最广的 shell 。 1977 年由 Stephen Bourne 在贝尔实验室编写。虽无明文规定，但已成为事实上的标准；
* 引入了 shell 通用的、基础的功能，例如管道、变量、条件判断、循环等；
* 全部类 Unix 系统，都至少有一个与 sh 兼容的shell；
* sh 一般位于 /bin/sh。目前大部分系统，/bin/sh 都是一个链接，指向一个兼容 sh 的、功能更丰富的 shell，如 bash；

### Bash

https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html

https://en.wikipedia.org/wiki/Bash_(Unix_shell)

> **Bash** is a [Unix shell](https://en.wikipedia.org/wiki/Unix_shell) and [command language](https://en.wikipedia.org/wiki/Command_language) written by [Brian Fox](https://en.wikipedia.org/wiki/Brian_Fox_(computer_programmer)) for the [GNU Project](https://en.wikipedia.org/wiki/GNU_Project) as a [free software](https://en.wikipedia.org/wiki/Free_software) replacement for the [Bourne shell](https://en.wikipedia.org/wiki/Bourne_shell).
>
> First released in 1989, it has been used as the default login shell for most [Linux](https://en.wikipedia.org/wiki/Linux) distributions.



* 即 bash，1989年由 Brian Fox 为 GNU 项目编写。
* 综合了 sh、csh、ksh 等各种 shell 的特性；
* 名字有双关含义，既是字面上的意思，用于替换 sh，也隐含为了 GNU 而 born again 的意思；
* 是 Mac OS X 和大部分 Linux 发行版的默认 shell；

《[Bash 脚本教程 - 阮一峰](https://wangdoc.com/bash/)》

### Zsh

https://www.zsh.org/

https://ohmyz.sh/

https://en.wikipedia.org/wiki/Z_shell

> The **Z shell** (**Zsh**) is a [Unix shell](https://en.wikipedia.org/wiki/Unix_shell) that can be used as an interactive login shell and as a [command interpreter](https://en.wikipedia.org/wiki/Command_line_interpreter) for [shell scripting](https://en.wikipedia.org/wiki/Shell_script).
>
> Zsh is an extended [Bourne shell](https://en.wikipedia.org/wiki/Bourne_shell) with many improvements, including some features of [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)), [ksh](https://en.wikipedia.org/wiki/KornShell), and [tcsh](https://en.wikipedia.org/wiki/Tcsh).

# GUI 和 CLI 如何切换？

切换到 tty1 ~ tty6 终端：`Ctrl + Alt + [F1 ~ F6]`

切换到 GUI：`Ctrl + Alt + [F7]`

# 参考

https://en.wikipedia.org/wiki/Shell_(computing)