---
title: "bash"
tags: Bash
---

> GNU's UNIX compatible shell

# 简介（Intro）

## 什么是 Shell ？

* 一个命令行解释器，用于将用户输入的命令转换为系统操作；
* Shell 既是交互式命令语言（CMD），也是脚本编程语言（Script）；
* Shell 有很多内置在其源代码中的命令。这些命令是内置的，所以 Shell 不必到磁盘上搜索它们，执行速度因此加快。不同的 Shell 内置命令有所不同；

## 有哪些 Shell ？

### GUI (Graphical User Interface)

* GNOME
* KDE
* ……

### CLI (Command Line Interface)

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

## GUI 和 CLI 如何切换？

切换到 tty1 ~ tty6 终端：`Ctrl + Alt + [F1 ~ F6]`

切换到 GUI：`Ctrl + Alt + [F7]`

# 启动（Invocation）

## 启动流程

1. /bin/login 程序首先会检查 `/etc/passwd` 文件，在这个文件里包含了用户名、密码和该用户的登录 shell，如 /bin/bash 。 /bin/login 在子进程里用 `execve` 调用了 /bin/bash 。
2. /bin/bash 读取 **启动文件** 并启动
3. /bin/bash 处理用户键入的命令
* 在执行磁盘上某个程序时，我们通常不会指定这个程序文件的绝对路径，比如要执行 `echo` 命令时，我们一般不会输入 `/bin/echo` ，而仅仅是输入 `echo` 。那为什么这样 bash 也能够找到 /bin/echo 呢？原因是 Linux 操作系统支持这样一种策略：shell 的一个环境变量 PATH 里头存放了程序的一些路径，当 shell 执行程序时会去这些目录下查找。
* `which` 作为 shell（这里特指 bash ）的一个内置命令，如果用户输入的命令是磁盘上的某个程序，它会返回这个文件的全路径。

## 启动文件
注意，如果以下启动文件中的任一个存在但是不可读取， bash 将报告一个错误。

### interactive shell

#### login shell

* 首先执行：`/etc/profile`
* 然后按顺序查找以下 3 个文件，并执行第一个找到的文件：
  * `~/.bash_profile`
  * `~/.bash_login`
  * `~/.profile`
* 退出时，执行：`~/.bash_logout`

#### no login shell

执行：`~/.bashrc`

### non-interactive shell

当 Bash 以非交互的方式启动时（例如运行一个 shell 脚本），它在环境中查找变量  `BASH_ENV` ，如果它存在则将它的值展开，使用展开的值作为一个文件的名称，读取并执行。 Bash 运作的过程就如同执行了下列命令：

```shell
if [ -n "$BASH_ENV" ]; then 
    . "$BASH_ENV"; 
fi
```

但是没有使用 `PATH` 变量的值来搜索那个文件名。

# 参数（Parameters）

## 位置参数（Positional Parameters）
## 特殊参数（Special Parameters）
## 环境变量（Shell Variables）

# 扩展（Expansion）

命令行的扩展是在拆分成词之后进行的。共有七种类型的扩展，常用的四种如下

## 花括号扩展（Brace Expansion）

用于偷懒，例如：

```
x{a,b,cd}y
```
扩展为：
```
xay xby xcdy
```

再例如：
```
mkdir /usr/local/src/bash/{old,new,dist}
```
扩展为：
```
mkdir /usr/local/src/bash/old
mkdir /usr/local/src/bash/new
mkdir /usr/local/src/bash/dist
```

## 参数和变量扩展（Parameter and Variable Expansion）

`${parameter}`

## 算术扩展（Arithmetic Expansion）

## 命令替换（Command Substitution）

# 别名（Aliases）

# 引用（Quoting）

引用用于...

* 阻止对特殊字符的处理。
* 阻止保留字被识别。
* 阻止参数的扩展。

三种引用机制：

| 引用符  | 描述                              |
| ---- | ------------------------------- |
| 转义字符 | 保留其后下一个字符的字面意义                  |
| 单引号  | 保留引用中所有字符的字面意义                  |
| 双引号  | 保留引用中所有字符的字面意义，例外的情况是 $, `, 和 \ |

单引号与双引号的使用区别：

![quoting](https://github.com/cynthia903/study-notes/blob/master/SA/GNU/assets/bash_quoting.png)

# 命令执行（Command Execution）

# 内建命令（Builtin Commands）

## 作业控制（Job Control）

Bash 是一个多任务的 CLI ，有以下作业控制相关的命令：

| 命令     | 描述                                       |
| ------ | ---------------------------------------- |
| jobs   | 显示（当前会话中的）后台作业表                          |
| fg     | 将后台作业调到前台执行（前台运行作业）                      |
| bg     | 继续执行指定的后台作业（后台运行作业）                      |
| Ctrl+Z | 暂停/挂起目前的命令，转入后台运行。通过在命令后追加一个&，可以将该命令转入后台运行 |
| Ctrl+C | 终止目前的命令                                  |

# 注释（Comments）

以 `#` 起始的词使得这个词和所有同一行上所有剩余的字符都被忽略。