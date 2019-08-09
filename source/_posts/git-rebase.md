---
title: "Git 实战系列（六）git rebase 命令衍合分支"
date: 2015-08-20 22:04:13
updated: 
tags: Git
---

本文介绍一个生僻但相当好用的命令 `rebase`（衍合）。

# 使用场景

衍合的两个使用场景：

1. 生成干净历史、补丁
2. 整理当前分支

## 生成干净历史

开发过程中，常常需要定期将最新的远程分支拉取（`pull`）到本地分支，保持本地代码最新（up to date）。如果拉取频繁，`pull` 默认的 `merge` 行为会造成祖先图谱（ancestry graph）无谓的复杂：

```
$ git pull origin master    // pull = fetch + merge

*   ab900eb - 三方合并版本（注意这里！）
|\
| * 756ba83 - 本地分支提交的版本
* | 915fe84 - 先被推送到远程分支的版本
|/
*   e7ce3f8 - 基准版本（共同祖先）
```

解决方案是改用 `rebase` 命令，其产生的祖先图谱如下，非常简洁：

```
*   dc6baf3 - 本地分支提交的版本（注意这个提交被改写了！）
|
*   915fe84 - 先被推送到远程分支的版本
|
*   e7ce3f8 - 基准版本（共同祖先）
```

可见，这个神奇的命令功能类似 `merge` ，但它避免了上述无谓的合并节点，从而产生一个更为整洁的提交历史。如果视察一个衍合过的分支历史，仿佛所有的提交都是在一根时间轴上先后进行的，尽管实际上它们原本是同时并行发生的。这么做的好处是，非常便于项目管理人员按时间轴进行**代码审查**。

### 命令用法

可以在 `pull` 时主动加上 `--rebase` 参数：

```bash
$ git pull --rebase origin master
```

甚至推荐将 `rebase` 设为 `pull` 命令的默认行为，从而应用于所有新建分支：

```bash
$ git config --global branch.autosetuprebase always
```

注意，对于应用上述命令前已存在的分支（例如 `master`），需要补充执行如下配置：

```bash
$ git config branch.master.rebase true
```

### 命令原理

 下面进一步介绍 `rebase` 命令的原理：

1. 把本地分支从上一次 `pull` 之后的变更暂存起来；
2. 恢复到上一次 `pull` 时的情况；
3. 合并远程分支的提交；
4. 最后再逐一合并刚暂存下来的本地提交（相当于重放一遍）。

## 生成干净补丁

使用衍合的另一个目的，是想要得到一个能在远程分支上干净应用的补丁 — 比如某些项目、或些分支你不是维护者，但想帮点忙的话，最好用衍合：先在自己的一个独立分支中进行开发，当准备向主项目提交补丁的时候，根据最新的 `origin/master` 进行一次 `git rebase` 衍合操作然后再提交，这样维护者就**不需要做任何整合工作**（实际上是把解决分支补丁同最新主干代码之间冲突的责任，化转为由提交补丁的人来解决。），只需根据你提供的仓库地址作一次**快进合并**，或者直接采纳你提交的补丁。

## 整理当前分支

衍合的另一个用法是整理当前分支，使用 `git rebase [-i | --interactive]` 命令。

首先选取提交范围，`e7ce3f8` 为当前分支的历史提交：

```
*   dc6baf3 - commit3
|
*   915fe84 - commit2
|
*   e7ce3f8 - commit1
```
```bash
$ git rebase -i e7ce3f8

  1 pick dc6baf3 本地分支提交的版本
  2 pick 915fe84 先被推送到远程分支的版本
  3
  4 # Rebase 1ff20826..61527529 onto 1ff20826 (2 commands)
  5 #
  6 # Commands:
  7 # p, pick = use commit
  8 # r, reword = use commit, but edit the commit message
  9 # e, edit = use commit, but stop for amending
 10 # s, squash = use commit, but meld into previous commit
 11 # f, fixup = like "squash", but discard this commit's log message
 12 # x, exec = run command (the rest of the line) using shell
 13 # d, drop = remove commit
 14 #
 15 # These lines can be re-ordered; they are executed from top to bottom.
 16 #
 17 # If you remove a line here THAT COMMIT WILL BE LOST.
 18 #
 19 # However, if you remove everything, the rebase will be aborted.
 20 #
 21 # Note that empty commits are commented out
```

可见，我们可以选取、编辑、合并、丢弃指定提交，达到整理分支的目的。

# 使用风险

注意，衍合必须遵守的准则：**一旦本地分支中的提交（commit）已经被推送到远程仓库，就千万不要对该分支进行衍合操作。**如果把衍合当成一种**在推送（`push`）代码之前**整理提交历史的手段，而且仅仅衍合那些**尚未推送**的本地提交，就没问题。如果衍合那些已经推送的提交，并且已经有人基于这些提交对象开展了后续开发工作的话，就会出现叫人沮丧的麻烦。

# 参考

* 《[Git-分支-分支的衍合#衍合的风险](https://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E8%A1%8D%E5%90%88#%E8%A1%8D%E5%90%88%E7%9A%84%E9%A3%8E%E9%99%A9)》
* 《[团队开发里频繁使用 git rebase 来保持树的整洁好吗?](http://segmentfault.com/q/1010000000430041)》