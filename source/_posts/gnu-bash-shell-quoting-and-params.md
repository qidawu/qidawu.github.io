---
title: GUN/Bash 系列（三）Shell 引用与参数总结
date: 2015-03-10 21:41:04
updated:
tags: GNU/Linux
typora-root-url: ..
---

# Shell 语法

## 引用（Quoting）

引用用于：

* 阻止对特殊字符的处理。
* 阻止保留字被识别。
* 阻止参数的扩展。

三种引用机制：

| 引用符       | 描述                                                  |
| ------------ | ----------------------------------------------------- |
| 转义字符 `\` | 保留其后下一个字符的字面意义                          |
| 单引号 `''`  | 保留引用中所有字符的字面意义                          |
| 双引号 `""`  | 保留引用中所有字符的字面意义，例外的情况是 $, `, 和 \ |

单引号与双引号的使用区别：

![quoting](/img/gnu-linux/bash_quoting.png)

注意，反引号 `` ` `` 与单引号 `''` 和双引号 `""` 作用不同，是用于命令替换（Command Substitution），详见《Shell 常用扩展总结》。

## 注释（Comments）

以 `#` 起始的词使得这个词和所有同一行上所有剩余的字符都被忽略。

# Shell 参数（Shell Parameters）

*参数（Parameter）*是存储值的实体。它可以是以下三类：

* 变量
* 位置参数
* 特殊参数

## 变量（Varialbe）

变量，即用*名称（name）*表示的参数，其具有*值（value）*以及零或多个*属性（attributes）*。

* 通过 `$name` 引用，在双引号 `""` 中可以被引用。
* 通过以下语句为变量赋值：`name=[value]`。如果变量未赋值，默认值为 `null` 字符串。
* 通过内建命令 `unset` 为取消变量。

* 通过内建命令 `declare` 为变量分配*属性（attributes）*。

所有值都接受以下扩展：

* tilde expansion
* parameter and variable expansion
* command substitution
* arithmetic expansion
* quote removal

## 位置参数（Positional Parameters）

`$n`：`$1` 表示第一个参数，`$2` 表示第二个参数，以此类推。

## 特殊参数（Special Parameters）

`$0`：表示脚本文件名

`$#`：表示命令行参数的个数

`$?`：前一个命令或函数的返回码，`0` 为成功，非 `0` 为错误/失败

`$*`：以"参数1 参数2 ... " 的形式保存所有参数

`$@`：以"参数1" "参数2" ... 的形式保存所有参数

`$$`：本程序的 PID（进程 ID 号）

`$!`：最近执行的后台（即异步）命令的 PID

# 参考

https://www.gnu.org/software/bash/manual/html_node/Quoting.html

https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Shell-Parameters