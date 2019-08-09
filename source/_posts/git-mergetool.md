---
title: "Git 实战系列（九）git mergetool 工具解决冲突"
date: 2015-08-27 15:35:15
updated: 2015-09-11 15:35:15
tags: Git
---

续[上文](/2015/08/25/git-resolving-conflicts/)。

有时候合并操作并不会如此顺利。如果在不同的分支中都修改了同一个文件的同一部分，Git 就无法干净地把两者合到一起。这种问题只能由人来裁决，解决冲突的办法无非是从冲突中二者选其一或者由你亲自整合到一起。

你完全可以手工编辑处理冲突，或者推荐使用图形化的外部合并与比较工具（mergetool）。

# mergetool 是什么？

> Merge tool is a GUI that steps you through each conflict, and you get to choose how to merge. Sometimes it requires a bit of hand editing afterwards, but usually it's enough by itself. It is much better than doing the whole thing by hand certainly.

# mergetool 如何选择？

**mergetool 需自行选择安装。**选择很多，例如：meld, opendiff, kdiff3, tkdiff, xxdiff, tortoisemerge, gvimdiff, diffuse, ecmerge, p4merge, araxis, vimdiff, emerge ...

推荐使用 [meld](http://meldmerge.org/)，一款优秀的可视化 diff 和代码合并工具（merge tool），支持特性如下：

* 跨平台，支持 Linux/Unix、 OS X、Windows，多种便捷的安装方式
* 跨工具，支持多种版本控制系统（VCS），如 Git、SVN、Mercurial ...
* 支持双方或三方文件、目录对比
* GUI 界面好看 :)

# meld 如何使用？

安装好 meld ，还需进行如下配置：

## 用于 git diff

首先配置好 git ：

```bash
$ git config --global diff.external ~/meld.sh
```

然后准备编写 `meld.sh` 包装脚本：

```bash
$ vim ~/meld.sh
```

默认情况下， `git diff` 会传递 7 个参数给该包装脚本：

```bash
path old-file old-hex old-mode new-file new-hex new-mode
```

但我们仅仅只需要 `old-file` 和 `new-file` 参数，因此需要用包装脚本来传递它们。脚本内容如下：

```bash
#!/bin/sh
meld $2 $5
```

如果对涉及到的参数感兴趣，可以在脚本补充一段 `echo $0 $*`。

最后对于 Linux/Unix、OS X，还需要增加脚本的可执行权限：

```bash
$ chmod +x ~/meld.sh
```

以上配置好后，就可以调用图形化工具愉快的使用 `git diff` 了。

## 用于 git mergetool

首先配置好 git ：

```bash
$ git config --global merge.tool meld
```

如果合并的时候出现如下冲突：

```bash
CONFLICT (content): Merge conflict in index.html Automatic merge failed; fix conflicts and then commit the result.
```

使用如下命令：

```bash
$ git mergetool
```

就可以调用图形化工具愉快的解决冲突了。

在解决了所有文件的所有冲突后，运行 `git add` 将把它们标记为已解决状态即可（一旦暂存，就表示冲突已经解决）。

# 参考

* 《[How do I fix merge conflicts in Git?](http://stackoverflow.com/questions/161813/fix-merge-conflicts-in-git)》
* 《[Git Mergetool – Merging With a GUI](http://www.gitguys.com/topics/merging-with-a-gui/)》
* 《[外部的合并与比较工具](http://git-scm.com/book/zh/v1/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E9%85%8D%E7%BD%AE-Git#外部的合并与比较工具)》