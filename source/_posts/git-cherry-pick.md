---
title: "git cherry-pick 命令选取合并"
date: 2015-08-21 22:04:13
updated: 
tags: Git
---

`cherry-pick` 这个命令的名字是比较形象的，即“摘樱桃”，使用该命令可以将任意的 commit 合并到你想要的分支上。 例如：

```bash
# 切换到 master 分支
$ git checkout master

# cherry-pick 特性分支上的三个 commit
$ git cherry-pick e7ce3f8 915fe84 dc6baf3
```

合并完毕后，会在 master 分支上新产生三个 commit 号，但提交内容不变。

如果只是想[整理当前分支](/2015/08/20/git-rebase/)，可以使用 rebase 命令。

# 参考

《[Git知识总览(四) git分支管理之rebase 以及 cherry-pick相关操作](http://www.cnblogs.com/ludashi/p/8116434.html)》