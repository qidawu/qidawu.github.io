---
title: git merge 命令合并分支
date: 2015-08-17 22:48:24
updated:
tags: Git
---

# 快进式合并

当我们 `git pull` 拉取代码时，背后实际上是进行了一次“快进式合并”：

```bash
$ git fetch origin master:master
$ git merge origin/master

*   756ba83 - 特性分支版本2（SHA-1 不变）
|
*   915fe84 - 特性分支版本1（SHA-1 不变）
|
*   e7ce3f8 - master 基准版本（祖先）
```

那么，究竟什么是“快进式合并（fast-forward merge）”？如果顺着一个分支走下去可以直接到达另一个分支的话，那么 Git 在合并两者时，只会简单地把指针右移，因为这种单线的历史分支不存在任何需要解决的分歧，所以这种合并过程可以称为快进（Fast forward）。

快进式合并一般只用于同一分支内的代码合并。

# 非快进式合并

作为对比，如果加上 `--no-ff` 参数进行“非快进式合并（no-fast-forward merge）”，其祖先图谱如下：

```bash
$ git checkout master
$ git merge --no-ff feature-test

*   ab900eb - 合并版本（注意这里！）
|\
| * 756ba83 - 特性分支版本2
| * 915fe84 - 特性分支版本1
|/
*   e7ce3f8 - master 基准版本（祖先）
```

可见，合并后保留有分支历史痕迹，能看得出来曾经做过分支合并。为了保证版本演进的清晰，当分支合并回主干时（如上），建议都加上 `--no-ff` 参数。



