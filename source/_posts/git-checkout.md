---
title: "Git 实战系列（四）git checkout 命令撤销修改"
date: 2015-08-15 15:14:54
updated:
tags: Git
---

`checkout` 命令可以用于三种场景：

* 切换分支
* 创建分支
* 撤销修改

本文只介绍第三种场景。

# 例子

如果我们想要撤销一个文件的本地修改，自然可以手工编辑恢复，但这样做实在是吃力不讨好。 `checkout` 命令可以帮助我们：

## 只撤销本地修改

修改文件后，使用 `status` 命令查看一下文件状态：

```bash
$ git status
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

      modified:   /there/is/a/modified/file
```

Git 提示我们，对于未 `add` 进暂存区的文件，可以使用 `git checkout -- <file>` 快速撤销本地修改。

## 同时撤销本地和暂存区修改

那么，对于已 `add` 进暂存区的文件，如何撤销本地修改？还是先使用 `status` 命令查看一下文件状态：

```bash
$ git status
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

      modified:   /there/is/a/modified/file
```

### 先取消暂存修改

Git 提示我们，可以使用 `reset` 命令取消暂存：

```
$ git reset /there/is/a/modified/file
```

取消暂存后，文件状态就回到了跟“例1”一样了：

```bash
$ git status
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

      modified:   /there/is/a/modified/file
```

### 再撤销本地修改

这时按提示使用 `checkout` 即可：

```bash
$ git checkout -- /there/is/a/modified/file
```

这时工作目录就干净了：

```bash
$ git status
nothing to commit, working directory clean
```

可以看到，结合使用 `reset` 和 `checkout` 命令，可以撤销 index 和 working tree 的修改。

### 一步到位

那么有更便捷的、一步到位的办法吗？有，指定提交即可：

```bash
$ git status
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

      modified:   /there/is/a/modified/file
```

```bash
$ git checkout HEAD -- /there/is/a/modified/file
```

```bash
$ git status
nothing to commit, working directory clean
```

那么 `checkout` 命令的全貌究竟是怎样的呢？

# `checkout` 命令格式

`checkout` 命令的格式及描述如下：

```bash
git checkout [<tree-ish>] [--] <paths>...

Updates the named paths in the working tree from the index file (default) or from a named <tree-ish> (most often a commit, tag or branch)
```

* 默认使用 index 暂存区的内容覆盖本地修改，如果不指定 `<tree-ish>` 参数。
* 或者可以使用指定的提交、标记、分支版本覆盖本地修改。
* 为了避免文件路径 `<paths>` 和 `<tree-ish>` 同名而发生冲突，在 `<paths>` 前用 `--` 作为分隔。

# `checkout` 与 `reset`

还记得在《[git reset 命令回退版本](http://www.cnblog.me/2015/08/09/git-reset/)》中介绍的 `reset` 命令吗？它与 `checkout` 命令之间有什么区别与关系？

## 区别

在这里介绍 `reset` 命令的另一种形式：

```bash
git reset [<tree-ish>] [--] <paths>...

This form copy entries from <tree-ish> to the index for all <paths>. (It does not affect the working tree or the current branch.)
```

与 `checkout` 命令的参数一模一样，区别是什么？

| 命令         | 操作目标               | 描述       |
| ---------- | ------------------ | -------- |
| `checkout` | 工作目录（working tree） | 用于撤销本地修改 |
| `reset`    | 暂存区（index）         | 只用于覆盖暂存区 |

因此 `git reset <paths>` 等于 `git add <paths>` 的逆向操作。

如果企图用 `reset` 命令覆盖工作目录，是会报错的：
```
$ git reset --hard /there/is/a/modified/file
fatal: Cannot do hard reset with paths.
```

## 关系

> After running `git reset <paths>` to update the index entry, you can use `git checkout -- <paths>` to check the contents out of the index to the working tree. 
>
> Alternatively, using `git checkout [<tree-ish>] [--] <paths>` and specifying a commit, you can copy the contents of a path out of a commit to the **index** and to the **working tree** in one go.

# 参考

《[git checkout](http://git-scm.com/docs/git-checkout/)》
《[git reset](http://git-scm.com/docs/git-reset/)》
《[Git 教程 - 撤销修改](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374831943254ee90db11b13d4ba9a73b9047f4fb968d000)》