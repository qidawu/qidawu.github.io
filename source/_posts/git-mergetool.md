title: "git mergetool 工具解决冲突"
date: 2015-08-07 15:35:15
updated: 
tags: Git
---

续上文。

有时候合并操作并不会如此顺利。如果在不同的分支中都修改了同一个文件的同一部分，Git 就无法干净地把两者合到一起。这种问题只能由人来裁决，解决冲突的办法无非是从冲突中二者选其一或者由你亲自整合到一起。

可以手工编辑处理冲突，或者推荐使用图形化工具 mergetool 。

# mergetool 是什么？

> Merge tool is a GUI that steps you through each conflict, and you get to choose how to merge. Sometimes it requires a bit of hand editing afterwards, but usually it's enough by itself. It is much better than doing the whole thing by hand certainly.

# mergetool 如何选择？

**mergetool 需自行选择安装。**选择很多，例如：meld, opendiff, kdiff3, tkdiff, xxdiff, tortoisemerge, gvimdiff, diffuse, ecmerge, p4merge, araxis, vimdiff, emerge ...

我的选择是 [meld](http://meldmerge.org/)，

# mergetool 如何使用？

安装好心仪的 mergetool 后，使用如下命令配置，例如使用 meld：

```bash
git config --global merge.tool meld
```

如果合并的时候出现如下冲突：

```bash
CONFLICT (content): Merge conflict in index.html Automatic merge failed; fix conflicts and then commit the result.
```

使用如下命令：

```bash
git mergetool
```

调用图形化工具愉快的解决冲突。

在解决了所有文件的所有冲突后，运行 `git add` 将把它们标记为已解决状态即可（一旦暂存，就表示冲突已经解决）。

# 参考

* 《[How do I fix merge conflicts in Git?](http://stackoverflow.com/questions/161813/fix-merge-conflicts-in-git)》
* 《[Git Mergetool – Merging With a GUI](http://www.gitguys.com/topics/merging-with-a-gui/)》