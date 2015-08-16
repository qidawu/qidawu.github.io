title: "git checkout 命令撤销修改"
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

## 只撤销本地修改

如果我们想要撤销一个文件的本地修改，自然可以手工编辑恢复，但这样做实在是吃力不讨好。

如果使用 `status` 命令查看一下状态：

```bash
$ git status
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  
      modified:   /there/is/a/modified/file
```

会发现 Git 提示我们，对于未 `add` 进暂存区的文件，可以使用 `git checkout -- <file>` 快速撤销本地修改。

## 同时撤销本地和暂存区修改

那么，对于已 `add` 进暂存区的文件，如何快速撤销本地修改？指定所需版本即可，例如：`git checkout HEAD -- <file>` 。

那么 `checkout` 命令的全貌是究竟怎样的呢？

# `checkout` 命令格式

`checkout` 命令的格式及描述如下：

```bash
git checkout [<tree-ish>] [--] <paths>...

Updates the named paths in the working tree from the index file (default) or from a named <tree-ish> (most often a commit, tag or branch)
```

* 默认使用 index 暂存区的内容覆盖本地修改，如果不指定 `<tree-ish>` 参数。
* 或者可以使用指定的提交、标记、分支版本覆盖本地修改。
* 为了避免文件路径 `<paths>` 和 `<tree-ish>` 同名而发生冲突，可以在 `<paths>` 前用 `--` 作为分隔。

# `checkout` 与 `reset` 对比

扩展 《git reset》 。

http://git-scm.com/docs/git-reset/

```
$ git reset --hard /there/is/a/modified/file
fatal: Cannot do hard reset with paths.
```

# 参考

《[Git 教程 - 撤销修改](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374831943254ee90db11b13d4ba9a73b9047f4fb968d000)》

《[git reset](http://git-scm.com/docs/git-reset/)》