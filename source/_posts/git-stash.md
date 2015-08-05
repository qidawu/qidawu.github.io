title: git stash 命令解决冲突
date: 2015-08-05 22:56:06
updated:
tags: Git
---

> Stashing takes the dirty state of your working directory – that is, your modified tracked files and staged changes – and saves it on a stack of unfinished changes that you can reapply at any time.

stash 命令可以将“修改过的被追踪的文件（modified tracked files）”和“暂存的变更（staged changes）”暂存到临时堆栈中，并将工作目录还原干净。该命令可以用于两种场景：

* [暂存当前分支的修改，以便切换分支](http://git-scm.com/book/zh/v1/Git-工具-储藏（Stashing）)
* 解决冲突

# 解决冲突

如果工作目录的代码做了改动但尚未提交，拉取远程仓库的新提交时，往往会提示冲突：

```
$ git pull
error: Your local changes to the following files would be overwritten by merge:
        /there/is/a/conflict/file
Please, commit your changes or stash them before you can merge.
```

## commit 处理

把改动提交到本地仓库，再次 `git pull` 拉取远程仓库的代码，如无冲突，会自动产生一次“合并”提交：

```
$ git lg
*   e6f4e18 - Merge branch 'master' of origin (1 minutes ago)
|\  
* | abfa93b - 本地仓库的提交 (2 minutes ago)
| * 1f1c21d - 远程仓库的提交 (3 minutes ago)
|/  
* 17ef24c - 基准版本 (4 minutes ago)

```

这是因为 Git 的默认策略是“快进式合并（fast-farward merge）”。当没法按时间轴快进时，当然只能做三方合并了。

如果本地仓库的提交一直不 `push` 到远程仓库，极端情况下每一次 `pull` 都可能会产生一次“合并”提交，这会造成线图（graph）无谓的复杂，这时会推荐使用 rebase 避免无所谓的合并节点：

```
$ git pull --rebase
```

rebase 的详细原理在此暂且不表。

## stash 处理

推荐使用 stash 处理，所有合并都在工作目录中完成，不会产生“合并”提交。

```
$ git stash
Saved working directory and index state ......
```

```
$ git status
On branch master
nothing to commit, working directory clean
```

可以看到工作目录干净了，`pull` 命令将能够拉取并顺利合并：

```
$ git pull
```

重新还原暂存：

```
$ git stash pop
```

如果有冲突，手工编辑解决并再次跟踪即可：

```
$ git add .
```