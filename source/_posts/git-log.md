title: "git log 查看提交历史"
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

如果对输出格式还不满意，可以使用 `--pretty` 参数定制输出格式。但由于该参数的选项较多，推荐设置为别名（alias）使用：

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

## 筛选提交历史

当某个特性分支开发完成之后，我们想要筛选并看清将要合并到主干的是哪些代码，从而理解它们到底做了些什么，是否真的要并入。可以用 `--not` 选项屏蔽 `master` 分支，这样就会剔除重复的提交历史，看起来更清晰：

```bash
$ git log feature-cache --not origin/master
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