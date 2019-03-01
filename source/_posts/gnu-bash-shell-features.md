---
title: GUN/Bash 系列（三）Shell 特性
date: 2015-11-13 21:41:04
updated:
tags: GNU/Linux
---

# Shell 语法

## 引用（Quoting）

引用用于：

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

![quoting](/img/gnu-linux/bash_quoting.png)

## 注释（Comments）

以 `#` 起始的词使得这个词和所有同一行上所有剩余的字符都被忽略。

# Shell 命令

## 简单命令（Simple Commands）

## 管道（Pipelines）

pipeline（管道）是一个或多个命令的序列，用字符 | 分隔。管道的命令格式如下：

```bash
command [ | command2 ... ]
```

管道的特点如下：

* 管道是一个由“标准输入输出”链接起来的进程集合；
* 管道中的每个命令都作为单独的进程来执行（即在一个子 shell 中启动）；
* 每一个进程的输出（stdout）被直接作为下一个进程的输入（stdin）；
* 管道命令不处理 standard error output（stderr）；
* 管道的符号为：`|`

管道的处理流程如下图：

![pipe](/img/gnu-text-utilities/pipe.png)

## 重定向（Redirection）

在命令执行前，它的输入和输出可能被 redirected (重定向)，该功能可以用于如下场景：

* 屏幕输出的信息很重要，而且需要将它存下来时；
* 一些运行命令的可能已知错误信息，想以 `2> /dev/null` 将它丢掉；
* 一些系统的例行命令（例如写在 `/etc/crontab`）的运行结果，需要存下来时；
* 错误信息与正确信息需要分别输出时。

例子：

快速创建带内容的文件

```bash
$ echo "hello world" > /tmp/file
$ cat /tmp/file
```

将已知错误信息丢弃

```bash
$ find /tmp/ -name 2> /dev/null
```

### 描述符（Descriptor Number）

| 描述符 | 描述                   |
| ------ | ---------------------- |
| `0`    | 标准输入（stdin）      |
| `1`    | 标准输出（stdout）     |
| `2`    | 标准错误输出（stderr） |

### 操作符（Operator）

| 操作符 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| `<`    | 重定向输入（Redirecting Input）                              |
| `>`    | 重定向输出（Redirecting Output），与 `1>` 等价               |
| `>>`   | 追加到重定向输出（Appending Redirected Output）              |
| `2>`   | 重定向错误输出（Redirecting Error）                          |
| `2>>`  | 追加到重定向错误输出（Appending Redirected Error）           |
| `&>`   | 重定向标准输出和标准错误输出（Redirecting Standard Output and Standard Error）。 **推荐使用**，它与 `>word 2>&1` 在语义上等价 |
| `>&`   | 同上，但不推荐使用                                           |
| `2>&1` | 将标准错误输出重定向到标准输出                               |

## 序列（Lists of Commands）

list（序列）是一个或多个管道，用操作符 `;`、`&`、`&&`、`||` 分隔的序列, 并且可以选择用 `;`、`&`、`<newline>` 新行符结束。

| 操作符        | 例子                   | 描述                                                         |
| ------------- | ---------------------- | ------------------------------------------------------------ |
| `&&`          | command1 && command2   | 一个 AND 序列。command2 只有在 command1 返回 0 时才被执行    |
| <code>&#124;&#124;</code>       | command1 &#124;&#124; command2 | 一个 OR 序列。command2 只有在 command1 返回非 0 状态时才被执行 |
| `;`           | command1; command2     | 结束一个序列。不考虑命令的退出状态，连续执行命令             |
| `<newline>` | command<newline\>     | 结束一个序列                                                 |
| `&`           | command1 &             | 如果一个命令是由 & 结束的, shell 将在后台的子 shell 中执行这个命令 |

* AND 和 OR 序列的返回状态是序列中最后执行的命令的返回状态。
* 这些序列操作符中， `&&` 和 `||` 优先级相同， `;` 和 `&` 优先级相同。

### 退出状态（Exit Status）

* 从 shell 的角度看，一个命令退出状态是 0 意味着成功退出。 非零状态值表明失败。
* 如果没有找到命令，为执行它而创建的子进程返回 127。
* 如果找到了命令但是文件不可执行，返回状态是 126。
* 如果命令由于扩展或重定向错误而失败，退出状态大于零。

### 使用场景

在某些情况下，很多命令我想要一次输入去运行，有两种方法：

1. Shell Script
2. 使用序列

例如，一串无人值守源代码形式安装的命令如下：

```bash
$ ./configure && make && make install
```

## 复合命令（Compound Commands）

compound command（复合命令）是如下情况之一：

* `(list)`
* `{ list; }`
* `((expression))`
* `[[ expression ]]`

* `if list; then list; [ elif list; then list; ] ... [ else list; ] fi`
* `case word in [ [(] pattern [ | pattern ] `

* `while list; do list; done`
* `until list; do list; done`
* `for name [ in word ] ; do list ; done`
* `for (( expr1 ; expr2 ; expr3 )) ; do list ; done`

* `select name [ in word ] ; do list ; done`

# Shell 函数（Shell Functions）

# Shell 参数（Shell Parameters）

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

# Shell 扩展（Shell Expansions）

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