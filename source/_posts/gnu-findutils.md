---
title: find 命令使用总结
date: 2017-07-08 22:49:39
updated:
tags: GNU/Linux
---

`find` 是最常见和最强大的查找命令，直接查找磁盘，缺点耗时长。命令格式如下：

`find [path...] [expression]`

The  expression  is made up of :

* `options` (which affect overall operation rather than the processing of a specific file, and always return true)
* `tests` (which return a true or false value)
* `actions` (which have side effects and return a true or false value)

all separated by `operators`.

# 选项

常用选项：

* `-maxdepth 1` 只查找当前目录

# 条件

常用条件：

- `-name` 名称查找（例如：`find ./ -name 'struts*'`）
- `-type` 类型查找
  - `d` 目录类型
  - `f` 常规文件类型
  - `l` 软链类型
  - ……
- `-user` 设定所属用户的名称

- `-group` 设定所属用户组的名称

- `-perm` 设定权限

- `-regex` 使用正则表达式进行匹配

- `-size` 表示文件大小
- `-empty` 空文件或空目录
- `-atime / -amin` File was last accessed n*24 hours/n minutes ago.

- `-ctime / -cmin` File’s status was last changed n*24hours/n minutes ago.

- `-mtime / -mmin` File’s data was last modified n*24hours/n minutes ago.

# 动作

常用动作：

* `-print` 输出结果（默认动作）
* `-ls` 输出详情
* `-delete` 执行删除
* `-exec` 执行指定命令

# 操作符

操作符用于提高表达式的优先级，下列操作符的优先级以倒序排列：

- `( expr )` 强制最高优先级
- `! expr` 求反操作
- `expr1 expr2` (or `expr1 -a expr2`) 求与操作
- `expr1 -o expr2` 求或操作

# 例子

## 按文件名查找

查找当前目录树中，名字以 `fileA_` 或 `fileB_` 开头的所有文件：

```bash
$ find . -name 'fileA_*' -o -name 'fileB_*'
```

查找当前目录树中的 `foo.cpp` 文件，查找过程中排查掉 `.svn` 子目录树：

```bash
$ find . -name 'foo.cpp' '!' -path '.svn'
```

查找当前目录树中，以 `my` 开头的常规文件，并输出文件详情：

```bash
$ find . -name 'my*' -type f -ls
```

## 按大小查找

查找大小在 100k~500k 的文件：

```bash
$ find . -size +100k -a -size -500k
```

查找空文件：

```bash
$ find . -size 0k
```

查找非空文件：

```bash
$ find . ! -size 0k
```

## 删除文件或目录

删除空文件或空目录：

```bash
$ find . -type f -empty -delete
$ find . -type d -empty -delete
```

根据 inode 号删除乱码文件：：

```bash
$ find . -inum <inode-number> -exec rm -i {} \;
```

或

```bash
$ rm `find ./ -inum <inode-number>`
```

![find -exec](/img/gnu-linux/find.jpg)