---
title: Git 分支模型实践
date: 2016-04-05 08:48:24
updated: 2017-04-20 18:48:24
tags: Git
---

有了一套成熟的[分支模型](/2016/04/03/git-branch/)以及配套的[权限控制](/2016/04/04/git-permissions/)之后，接下来我们以一个例子来演示如何实践这套流程。

# 分支模型实践

## 创建版本分支

首先，项目管理员（Master）从 `master` 分支中创建出版本分支 `release-*` 进行新版本的开发，`*` 为发布日期：

```bash
$ git checkout -b release-20190101

do something and commit...

$ git push origin release-20190101
```

### 合并分支

版本开发完毕，Master 需要**整理**版本分支（例如从中挑选出能够发版的提交，剔除掉不能发版的提交），合并回 `master` 分支并进行发版。

### 标记新版本

发版完毕，Master 打 Tag 标记该新版本，以便后续回顾：

```bash
$ git tag release-20190101 -m "XX 项目 v1.0 版本"
$ git push origin release-20190101
```

注意，在默认情况下，`git push` 并不会把标签（tag）推送到远端仓库上，只有通过显式命令才能分享标签到远端仓库。其命令格式如同推送分支，运行 `git push origin [tagname]` 即可。如果要一次推送所有本地新增的标签上去，可以使用 `--tags` 选项。

### 清理分支

最后是一些清理工作，Master 需要删除已完成开发的版本分支，避免分支越来越多导致不好管理：

```bash
$ git branch -d release-20190101
$ git push --delete origin release-20190101
```

最后，列出所有远程和本地分支确认下：

```bash
$ git branch -a
```

## 创建特性分支

首先，开发人员（Developer）从 `master` 分支中创建出特性分支：

```bash
$ git checkout -b feature-test

do something and commit...

$ git push origin feature-test
```

### 定期合并

由于特性分支可能会跨版本开发，因此需要定期维护：主要的工作就是定期将 `master` 分支合并进来，保持同步。

### 决断代码

特性分支开发完成之后，如果想要筛选出将要被合并的提交有哪些，可以参考[这里](/2015/08/04/git-log/#筛选提交历史)。

# 总结

## 代码提交指南

* 请不要在更新中提交多余的白字符（whitespace）。Git 有种检查此类问题的方法，在提交之前，先运行 `git diff --check` ，会把可能的多余白字符修正列出来。
* 请将每次提交限定于完成一次逻辑功能。并且可能的话，适当地分解为多次小更新，以便每次小型提交都更易于理解。
* 最后需要谨记的是提交说明的撰写。可以理解为第一行的简要描述将用作邮件标题，其余部分作为邮件正文。

## 分支管理指南

* 主分支 `master` 一般不提交代码，只合并代码。
* 各特性分支要定期将 `master` 分支合并进来，避免后续处理合并请求时产生冲突，以减轻项目管理员的工作负担。
* 发版之后，项目管理员要记得打 tag 。

# 参考

* 《[分布式 Git](https://git-scm.com/book/zh/v1/%E5%88%86%E5%B8%83%E5%BC%8F-Git)》