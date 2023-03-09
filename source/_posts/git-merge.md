---
title: Git 实战系列（五）git merge 分支合并策略
date: 2015-08-17 22:48:24
updated:
tags: Git
typora-root-url: ..
---

# 快进式合并

默认情况下，当使用 `git merge` 合并代码时，背后实际上是进行了一次“快进式合并”：

```bash
$ git checkout master
$ git merge feature-test
```

什么是“快进式合并（fast-forward merge）”？如果顺着一个分支走下去可以直接到达另一个分支的话，那么 Git 在合并两者时，只会简单地把指针右移，因为这种单线的历史分支不存在任何需要解决的分歧，所以这种合并过程可以称为快进（Fast forward）。

![fast-forward merge](/img/git/fast-forward-merge.png)

# 非快进式合并

作为对比，加上 `--no-ff` 参数进行“非快进式合并（no-fast-forward merge）”：

```bash
$ git checkout master
$ git merge --no-ff feature-test
```

其祖先图谱如下：

![no-fast-forward merge](/img/git/no-fast-forward-merge.png)

可见，合并后保留有分支历史痕迹（每一次提交），能看得出来曾经做过分支合并，版本演进比较清晰。

# 压缩合并

但大多数时候，没有必要把特性分支的历史保留得太细，只需把整个特性分支压缩（squash）为主干上的一个提交即可。这样的祖先图谱既清晰，又能方便后人审查代码，推荐使用：

```bash
$ git checkout master
$ git merge --squash feature-test
$ git commit
```

# 参考

* 《[Git 分支管理策略 - 廖雪峰](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013758410364457b9e3d821f4244beb0fd69c61a185ae0000)》