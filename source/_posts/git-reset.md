title: "git-reset"
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

```
git reset [<mode>] [<commit>]
```

该命令用于重置工作目录、暂存区域的修改，或回退本地仓库的版本。

## mode 参数

mode 参数必须是以下五种的一种：

### `--soft`

HEAD Only

Git 本地仓库的版本回退速度之所以快，全因为 Git 在内部有个指向当前版本的 HEAD 指针，当你回退版本的时候，Git 仅仅是把 HEAD 指针往回移动。

### `--mixed`

HEAD and Index

默认参数。除了回退本地仓库的版本，还会重置暂存区域（也称为 Index 索引文件）。

这个默默无闻的 `--mixed` 参数其实很常见，每次运行 `git status` 时都会看到它的作用：

```
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   /there/is/a/new/file
        modified:   README.md
```

由于该命令实在太常用了，因此会被设为 alias 以便使用：

```
$ git config --global alias.unstage 'reset HEAD'
$ git unstage
```

### `--hard`

HEAD, Index, and Working Directory

终极武器，将包括工作目录在内的三个工作区域全部重置或回退，工作目录将重置干净，慎用。

```
$ git reset --hard
HEAD is now at ......

$ git status
nothing to commit, working directory clean
```

### `--merge`
### `--keep`

## commit 参数