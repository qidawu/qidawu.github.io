---
title: GNU/Bash 系列（二）Shell 启动流程
date: 2015-03-05 22:38:36
updated:
tags: GNU/Linux
---

了解完什么是 Shell，有哪些 Shell 类型之后，本文开始主要关注 Bash Shell。

# 启动流程

1. /bin/login 程序首先会检查 `/etc/passwd` 文件，在这个文件里包含了用户名、密码和该用户的登录 shell，如 /bin/bash 。 /bin/login 在子进程里用 `execve` 调用了 /bin/bash 。
2. /bin/bash 读取 **启动文件** 并启动
3. /bin/bash 处理用户键入的命令
* 在执行磁盘上某个程序时，我们通常不会指定这个程序文件的绝对路径，比如要执行 `echo` 命令时，我们一般不会输入 `/bin/echo` ，而仅仅是输入 `echo` 。那为什么这样 bash 也能够找到 /bin/echo 呢？原因是 Linux 操作系统支持这样一种策略：shell 的一个环境变量 PATH 里头存放了程序的一些路径，当 shell 执行程序时会去这些目录下查找。
* `which` 作为 shell（这里特指 bash ）的一个内置命令，如果用户输入的命令是磁盘上的某个程序，它会返回这个文件的全路径。

# 启动文件

## 交互式 shell

Bash 如何执行它的启动文件？交互式 shell（interactive shell）下需要区分两种情况：

### login shell

在下列情况下，我们可以获得一个 login shell：

* 登录系统时获得的顶层 shell，无论是通过本地终端登录，还是通过网络 `ssh` 登录。这种情况下获得的 login shell 是一个交互式 shell。
* 在终端下使用 `--login` 选项调用 bash，可以获得一个交互式 login shell。
* 在脚本中使用 `--login` 选项调用 bash（比如在 shell 脚本第一行做如下指定：`#!/bin/bash --login`），此时得到一个非交互式的 login shell。
* 使用 `su -` 切换到指定用户时，获得此用户的 login shell。如果不使用 `-`，则获得 non-login shell。

login shell 与 non-login shell 的主要区别在于它们启动时会读取不同的配置文件，从而导致环境不一样。login shell 启动时：

* 首先读取全局配置：`/etc/profile`
* 然后依次查找以下三个文件，读取第一个找到且可读的文件：
  * `~/.bash_profile`
  * `~/.bash_login`
  * `~/.profile`

* 退出时，读取：`~/.bash_logout`

### no login shell

交互式的 non-login shell 启动时，会读取：

* `~/.bashrc`

通常我们要定制一些配置时（例如 alias 别名），会将配置写在 `~/.bashrc` 中，然后在 `~/.bash_profile` 中读取 `~/.bashrc`，这样可以保证 login shell 和 non-login shell 得到相同的配置:

```bash
test -f ~/.bashrc && . ~/.bashrc
```

至于 `/etc/profile` 就不要轻易去改啦，毕竟会影响系统全局的配置。

## 非交互式 shell

* 当 Bash 以非交互的方式（non-interactive shell）启动时，**例如在运行一个 shell 脚本时**，它会查找环境变量 `BASH_ENV`，如果存在则将它的值展开，使用展开的值作为一个文件的名称，读取并执行。 Bash 运作的过程就如同执行了下列命令：

  ```bash
  if [ -n "$BASH_ENV" ]; then 
      . "$BASH_ENV"; 
  fi
  ```

* 其它情况：

  > Aliases are not expanded when the shell is not interactive.
  >
  > Functions are executed in the context of the current shell; no new process is created to interpret them (contrast this with the execution of a shell script).

# 参考

《[Bash Startup Files](http://www.gnu.org/software/bash/manual/bashref.html#Bash-Startup-Files)》

《[Interactive Shells](http://www.gnu.org/software/bash/manual/bashref.html#Interactive-Shells)》

《[使用$BASH_ENV来提权](http://linux.chinaunix.net/techdoc/develop/2008/09/16/1032346.shtml)》