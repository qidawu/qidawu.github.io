---
title: GUN/Bash 系列（六）Shell 常用扩展总结
date: 2015-03-16 21:41:04
updated:
tags: GNU/Linux
typora-root-url: ..
---

# Shell 扩展（Shell Expansions）

命令行的扩展是在拆分成词之后进行的。共有七种类型的扩展：

* brace expansion
* tilde expansion
* parameter and variable expansion
* command substitution
* arithmetic expansion
* word splitting
* filename expansion

常用的四种如下

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

```bash
${parameter}
```

## 算术扩展（Arithmetic Expansion）

```bash
$(( expression ))
```

## 命令替换（Command Substitution）

命令替换允许命令的输出替换命令本身。当命令被以下特殊字符包括时，将发生命令替换：

```bash
$(command)
```

或

```bash
`command`
```

# 参考

https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Shell-Expansions