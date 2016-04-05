title: Git 解决冲突
date: 2015-08-25 22:56:06
updated:
tags: Git
---

如果工作目录的本地代码做了修改但尚未提交，`pull` 拉取远程仓库的新提交时，往往会提示冲突：

```bash
$ git pull
error: Your local changes to the following files would be overwritten by merge:
        /there/is/a/conflict/file
Please, commit your changes or stash them before you can merge.
```

如上所示，有 `commit` 和 `stash` 两种处理方法。针对本地代码的**完成情况**我们需要作出选择。

# 代码已完成

如果确认本地代码已经**完成无误**，可以先将本地代码 `commit` 到本地仓库。再次 `pull` 拉取远程仓库时，如无冲突，Git 会自动产生一次“合并”提交：

```bash
$ git lg
*   e6f4e18 - Merge branch 'master' of origin (1 minutes ago)
|\  
* | abfa93b - 本地仓库的提交 (2 minutes ago)
| * 1f1c21d - 远程仓库的提交 (3 minutes ago)
|/  
* 17ef24c - 基准版本 (4 minutes ago)
```

这是因为 `pull` 的默认策略是“fetch + [merge](/2015/08/17/git-merge/)”。如果本地仓库的提交一直不 `push` 到远程仓库，极端情况下每一次 `pull` 都可能会产生一次“合并”提交，这会造成祖先图谱（graph）无谓的复杂。此时推荐使用 [rebase](/2015/08/20/git-rebase/) 避免本地仓库无谓的合并节点：

```bash
$ git pull --rebase
```

# 代码未完成

但如果本地代码**仍未完成**，此时推荐使用 `stash` 命令暂存修改，避免将未完成的功能代码 `commit` 到本地仓库，污染仓库。

## 暂存当前修改

第一步，`stash` 暂存当前修改：

```bash
$ git stash save "填写你的备注"
Saved working directory and index state ......
```

```bash
$ git status
On branch master
nothing to commit, working directory clean
```

可以看到暂存后工作目录一干二净。这是因为 `stash` 命令可以将“修改过的被追踪的文件（modified tracked files）”和“暂存的变更（staged changes）”暂存到临时堆栈中，并将工作目录还原干净，以便后续的操作。

> Stashing takes the dirty state of your working directory – that is, your modified tracked files and staged changes – and saves it on a stack of unfinished changes that you can reapply at any time.

## 拉取远程仓库

第二步，继续 `pull` 拉取远程仓库并进行自动合并：

```bash
$ git pull
```

## 重新还原暂存

第三步，`stash pop` 重新还原暂存修改：

```bash
$ git stash pop
```

## 处理冲突

最后一步，如果还原后产生冲突，需要[手工或使用 `mergetool`](/2015/08/27/git-mergetool/) 进行处理。处理完毕后，使用 `add` 标明冲突已解决：

```bash
$ git add .
```

# 参考

《[Git-工具-储藏（Stashing）](http://git-scm.com/book/zh/v1/Git-工具-储藏（Stashing）)》