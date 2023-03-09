---
title: "Git 实战系列（二）git log 查看提交历史"
date: 2015-08-04 12:08:21
updated: 2015-09-15 12:08:21
tags: Git
---

在提交了若干更新之后，又或者克隆了某个项目，想回顾下提交历史，可以使用 `git log` 命令查看，或者推荐使用 git 自带的图形化工具 `gitk`。

# 命令方式

## 定制输出格式 1

`git log` 的默认输出格式非常不便于查阅提交历史，使用时可以带上以下三个参数：

``` bash
$ git log --graph --oneline --decorate
*   e6f4e18 (HEAD, master) Merge branch 'master' of origin
|\  
* | abfa93b 本地仓库的提交
| * 1f1c21d (origin/master) 远程仓库的提交
|/  
*   17ef24c 基准版本
```

命令的输出形象地展示了提交历史，包括本地分支比远程分支领先了多少个提交版本。

## 定制输出格式 2

如果对输出格式还不满意，可以使用 `--pretty` 参数定制输出格式：

```bash
$ git log -5 --pretty=format:"%h - %an, %ar : %s"

f757dcd - wuqd, 20 hours ago : commit msg 5
5ca68df - wuqd, 13 days ago : commit msg 4
486b8d4 - wuqd, 3 weeks ago : commit msg 3
e58ae38 - wuqd, 3 weeks ago : commit msg 2
4830852 - wuqd, 3 weeks ago : commit msg 1
```

由于该参数的选项较多，推荐设置为别名（alias）使用：

``` bash
$ git config --global alias.lg log --graph --pretty=format:'%Cred%h%Creset - %s %Cgreen(%cr) %C(bold blue)<%an>'
$ git lg
*   e6f4e18 - Merge branch 'master' of origin (1 minutes ago) <Cyn>
|\  
* | abfa93b - 本地仓库的提交 (2 minutes ago) <Pete>
| * 1f1c21d - 远程仓库的提交 (3 minutes ago) <John>
|/  
*   17ef24c - 基准版本 (4 minutes ago) <Jerry>
```

格式化输出，代码着色，而且附上了作者、提交时间和祖先图谱。

## 求差集

```bash
$ git log HEAD --not origin/master
$ git log HEAD ^origin/master
$ git log origin/master..HEAD
```

也可用于筛选出准备 `push` 到远程仓库的提交，例如：

```bash
$ git log --graph --oneline --decorate HEAD --not origin/master
*   e6f4e18 (HEAD, master) Merge branch 'master' of origin
|  
*   abfa93b 本地仓库的提交
```

# 图形化方式

如果对输出格式还不满意，推荐使用 `gitk` 命令调用图形化工具查阅提交历史：

![gitk](http://git-scm.com/figures/18333fig0202-tn.png)

上半个窗口显示的是历次提交的分支祖先图谱，下半个窗口显示当前点选的提交对应的具体差异（其中右边 Patch 窗口显示当前提交的**文件列表**，左边 Diff 窗口显示每个文件的提交差异）。

# 参考

《[Git 基础 - 查看提交历史](http://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)》