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

# 语法（Shell Grammar）

## 简单命令（Simple Commands）
## 管道（Pipelines）

pipeline（管道）是一个或多个命令的序列，用字符 | 分隔。管道的格式如下：
```
command [ | command2 ... ]
```

管道的处理流程如下：

![pipelines](https://github.com/cynthia903/study-notes/blob/master/SA/GNU/assets/bash_pipelines.png)

* 管道是一个由标准输入输出链接起来的进程集合。
* 管道中的每个命令都作为单独的进程来执行（即在一个子 shell 中启动）。
* 每一个进程的输出（stdout）被直接作为下一个进程的输入（stdin）。
* 管道命令不处理 standard error output（stderr）。

### 重定向（Redirection）

#### 描述符（Descriptor Number）

|描述符|描述|
|---|---|
|0|标准输入（stdin）|
|1|标准输出（stdout）|
|2|标准错误输出（stderr）|

#### 操作符（Operator）

|操作符|描述|
|---|---|
|<|重定向输入（Redirecting Input）|
|>|重定向输出（Redirecting Output），与 `1>` 等价|
|>>|追加到重定向输出（Appending Redirected Output）|
|2>|重定向错误输出（Redirecting Error）|
|2>>|追加到重定向错误输出（Appending Redirected Error）|
|&>|重定向标准输出和标准错误输出（Redirecting Standard Output and Standard Error）。 **推荐使用**，它与 `>word 2>&1` 在语义上等价|
|>&|同上，但不推荐使用|
|2>&1|将标准错误输出重定向到标准输出|

#### tee 命令

![tee](https://github.com/cynthia903/study-notes/blob/master/SA/GNU/assets/bash_pipelines_tee.png)

* 用于**双向**重定向。
* 用于将数据流处理过程中的**某段结果**保存到文件。

#### 使用场景

* 屏幕输出的信息很重要，而且需要将它存下来时；
* 一些运行命令的可能已知错误信息，想以 `2> /dev/null` 将它丢掉；
* 一些系统的例行命令（例如写在 `/etc/crontab`）的运行结果，需要存下来时；
* 错误信息与正确信息需要分别输出时。

例子：

快速创建带内容的文件

```
echo "hello world" > /tmp/file
cat /tmp/file
```

将已知错误信息丢弃

```
find /tmp/ -name 2> /dev/null
```

## 序列（Lists）

list（序列）是一个或多个管道，用操作符 ;, &, &&, 或 || 分隔的序列, 并且可以选择用 ;, &, 或 \<newline\>新行符结束。

|操作符|例子|描述|
|---|---|---|
|&&|command1 && command2|一个 AND 序列。command2 只有在 command1 返回 0 时才被执行|
|\|\||command1 \|\| command2|一个 OR 序列。command2 只有在 command1 返回非 0 状态时才被执行|
|;|command1; command2|结束一个序列。不考虑命令的退出状态，连续执行命令|
|\<newline\>|command\<newline\>|结束一个序列|
|&|command1 &|如果一个命令是由 & 结束的, shell 将在后台的子 shell 中执行这个命令|

* AND 和 OR 序列的返回状态是序列中最后执行的命令的返回状态。
* 这些序列操作符中， && 和 || 优先级相同， ; 和 & 优先级相同。

### 退出状态（Exit Status）

* 从 shell 的角度看，一个命令退出状态是 0 意味着成功退出。 非零状态值表明失败。
* 如果没有找到命令，为执行它而创建的子进程返回 127。
* 如果找到了命令但是文件不可执行，返回状态是 126。
* 如果命令由于扩展或重定向错误而失败，退出状态大于零。

### 使用场景

在某些情况下，很多命令我想要一次输入去运行，有两种方法：

1. Shell Script
2. 使用序列

无人值守源代码形式安装

```
./configure && make && make install
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

### 算术求值（Arithmetic Evaluation）

### 条件表达式（Conditional Expressions）

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

|引用符|描述|
|---|---|
|转义字符|保留其后下一个字符的字面意义|
|单引号|保留引用中所有字符的字面意义|
|双引号|保留引用中所有字符的字面意义，例外的情况是 $, `, 和 \|

单引号与双引号的使用区别：

![quoting](https://github.com/cynthia903/study-notes/blob/master/SA/GNU/assets/bash_quoting.png)

# 命令执行（Command Execution）

# 内建命令（Builtin Commands）

## 作业控制（Job Control）

Bash 是一个多任务的 CLI ，有以下作业控制相关的命令：

|命令|描述|
|---|---|
|jobs|显示（当前会话中的）后台作业表|
|fg|将后台作业调到前台执行（前台运行作业）|
|bg|继续执行指定的后台作业（后台运行作业）|
|Ctrl+Z|暂停/挂起目前的命令，转入后台运行。通过在命令后追加一个&，可以将该命令转入后台运行|
|Ctrl+C|终止目前的命令|

# 提示符（Prompting）

## 常用提示符

在*交互执行*时， bash 在准备好读入一条命令时显示主提示符 `PS1`， 在需要更多的输入来完成一条命令时显示 `PS2`。 Bash 允许通过插入一些反斜杠转义的特殊字符来定制这些提示字符串，常用提示符如下：

|提示符|描述|
|---|---|
|\d|日期，格式是 "星期 月份 日" (例如，"Tue May 26")|
|\D{format}|*format* 被传递给 strftime(3)，结果被插入到提示字符串中； 空的 *format* 将使用语言环境特定的时间格式。花括号是必需的|
|\t|当前时间，采用 24小时制的 HH:MM:SS 格式|
|\T|当前时间，采用 12小时制的 HH:MM:SS 格式|
|\@|当前时间，采用 12小时制上午/下午 (am/pm) 格式|
|\A|当前时间，采用 24小时制上午/下午格式|
|||
|\h|主机名，第一个 . 之前的部分|
|\H|主机名|
|\j|shell 当前管理的作业数量|
|\l|shell 的终端设备名的基本部分|
|\s|shell 的名称， $0 的基本部分（最后一个斜杠后面的部分）|
|\u|当前用户的用户名|
|\w|当前工作目录|
|\W|当前工作目录的基本部分|
|\!|此命令的历史编号|
|\#|此命令的命令编号 |
|\$|如果有效 UID 是 0，就是 #, 其他情况下是 $|

## 与提示符相关的环境变量

PS1

> 这个参数的值被扩展用作主提示符字符串。

PS2

> 这个参数的值同 PS1 一起被扩展，用作次提示符字符串。默认值是 `>`

PS3

> 这个参数的值被用作内建命令 select 的提示符

PS4

> 这个参数的值同 PS1 一起被扩展，在执行跟踪中在 bash 显示每个命令之前显示。

CentOS 默认的 PS1 (`[\u@\h \W]\$`) 与 PS2 (`>`)

![centos_prompting](https://github.com/cynthia903/study-notes/blob/master/SA/GNU/assets/bash_centos_prompting.png)

# 注释（Comments）

以 `#` 起始的词使得这个词和所有同一行上所有剩余的字符都被忽略。