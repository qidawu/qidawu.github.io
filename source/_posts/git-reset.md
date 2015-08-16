title: "git reset 命令回退版本"
date: 2015-08-09 22:11:03
updated: 
tags: Git
---

Git 相比 SVN 的其中一个卓越之处，就在于有各种“后悔药”可吃。其中一种“后悔药”叫做 `reset` 命令，相当好用。

# 三个工作区域

理解 `reset` 命令的前提是理解文件流转的三个工作区域：

* 工作目录（working directory）
* 暂存区域（staging area）
* 本地仓库（repo）

![](https://git-scm.com/figures/18333fig0106-tn.png)

# 命令

`reset` 命令有三种参数形式，本文只介绍最常用的一种：

```bash
git reset [<mode>] [<commit>]

Reset the current branch head (HEAD) to <commit>, optionally modifying index and working tree to match.
```

该命令用于回退本地仓库当前分支下的版本，并可以选择重置暂存区域、工作目录的修改。

## mode 参数

mode 参数必须是以下五种中的一种：

### `--soft`

> HEAD Only

Git 本地仓库的版本回退速度之所以快，全因为 Git 在内部有个指向当前版本的 HEAD 指针，当你回退版本的时候，Git 仅仅是把 HEAD 指针往回移动。

### `--mixed`

> HEAD and Index

默认参数。除了回退本地仓库的版本，还会重置暂存区域（也称为 Index File 索引文件）。

这个默默无闻的 `--mixed` 参数其实很常见，每次运行 `git status` 时都会看到它的作用：

```bash
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   /there/is/a/new/file
        modified:   README.md
```

由于该命令实在太常用了，因此会被设为 alias 以便使用：

```bash
$ git config --global alias.unstage 'reset HEAD'
$ git unstage
```

### `--hard`

> HEAD, Index, and Working Directory

终极武器，将包括工作目录在内的三个工作区域全部重置或回退，工作目录将重置得一干二净，慎用。

常见的做法是回退到上一个版本，连同工作目录，就像一切从未发生过一样：

```bash
$ git reset --hard HEAD~1
HEAD is now at ......

$ git status
nothing to commit, working directory clean
```

如果“回退前的版本”已经 `push` 到远程仓库，则不建议这么做。

### `--merge`

待补充。

### `--keep`

待补充。

## commit 参数

commit 参数有三种常见形式：

### SHA1

使用 SHA1 值回退到指定的版本，适用于 SH1 值已知的情况：

```bash
$ git reset 17ef24c
```

### HEAD

更常用的参数，适用于偷懒：

* `HEAD` 表示当前版本（默认参数）。
* 上一个版本为 `HEAD^` ，上上一个版本为 `HEAD^^` ，以此类推。
* 上 100 个版本，简写为 `HEAD~100` 。
* `ORIG_HEAD` 表示上一个 `HEAD` ，一般用于撤销上一次 `reset` 。（"reset" copies the old head to .git/ORIG_HEAD）

### HEAD@{}

Git 在 1.8.5 版本之后，加入了 `HEAD@{}` 功能，它通过一个链表记录 `HEAD` 的移动路径，链表头部的 `HEAD@{0}` 即 `HEAD` 指针。这个功能可以用于回退到一个早已忘记的提交。

这个功能一般配合 `reflog` 命令使用。

# 例子

更多例子参见 `git help reset` 的 EXAMPLES 部分。