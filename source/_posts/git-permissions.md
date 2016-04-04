---
title: Git 权限控制
date: 2016-04-04 15:18:17
updated:
tags: Git
---

除了 Git 命令，权限控制也是 Git 中极为重要的组成部分，本文主要介绍 GitLab 系统提供的最常用的权限控制功能。

# 分配成员角色

首先来了解下，Git 中的五种角色：

| 角色        | 描述         |
| --------- | ---------- |
| Owner     | Git 系统管理员  |
| Master    | Git 项目管理员  |
| Developer | Git 项目开发人员 |
| Reporter  | Git 项目测试人员 |
| Guest     | 访客         |

每一种角色所拥有的权限都不同，如下图：

![Git 权限控制](/img/git/git_permissions.png)

我们需要做的是，为项目成员分配恰当的角色，以限制其权限。

# 锁定受保护分支

在对 Git 不熟悉的时候，时常苦恼于各个分支不受约束，任何开发人员都可以向任何分支直接推送任何提交，各种未经审查的代码、花样百出的 Bug 就这样流窜在预发布分支上。

其实我们可以通过 GitLab 的**受保护分支（Protected Branches）**功能解决该问题，该功能可用于：

* 阻止 Master 角色以外的开发人员直接向此类分支推送代码，保持稳定分支的安全性；
* 在向受保护分支合并代码前，强制进行代码审查。

接下来我们就使用这项功能，锁定我们的受保护分支——主分支 `master` 和预发布分支 `release-*`，以阻止 Developer 直接向这两类分支中推送代码：

![Git 受保护分支](/img/git/git_protected_branches.png)

锁定后，Developer 推送代码将会报错：

```
$ git push origin master
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 283 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 1 (delta 0)
remote: GitLab: You are not allowed to access master!
remote: error: hook declined to update refs/heads/master
To git@website:project.git
 ! [remote rejected] master -> master (hook declined)
error: failed to push some refs to 'git@website:project.git'
```

# 发起合并请求

锁定受保护分支后，要么 Master 需要时刻、主动关注各特性分支的进度，要么 Developer 需要线下、口头向 Master 汇报其特性分支的进度，这两种做法都非常不便于 Master 管理每个预发布分支的合并，尤其在团队大、分支多的情况。

我们可以通过 GitLab 的**发起合并请求（Merge Request）**功能解决该问题，这样既可以让 Developer 更自如的掌控自己分支进度，在必要的时候才主动发起合并请求；又可以减轻 Master 的合并工作量和沟通成本，可谓一举两得。

## 新建合并请求

第一步，按表单要求填写合并请求。注意，对于 Developer 而言：

* `From` 是你的特性分支 `feature-*`；
* `To` 只可能是预发布分支 `release-*`；
* `Title` 和 `Description` 要填写恰当的分支描述；
* `Assign to` 是该项目的 Master。

![新建合并请求](/img/git/git_new_merge_request.png)

## 审查合并请求

第二步，Master 收到合并请求后，进行代码审查。逐一查看 `Commits` 一栏提交的内容即可，对于需要改进的代码，可以直接在该行添加注释，非常方便。

![接受合并请求](/img/git/git_accept_merge_request.png)

如果对整个请求还有疑问的地方，还可以通过底部的 `Discussion` 功能进行线上讨论。

## 处理合并请求

第三步，针对审查结果进行相应处理：

### 关闭

对于完全不合格的垃圾代码、或者废弃的特性分支的合并请求，Master 点击右上角的 `Close` 按钮即可。合并请求将被关闭，相当于扔进回收站。

### 改进

对于分支内需改进的代码，Developer 直接修正并推送即可，合并请求将会自动包含最新的推送提交。

### 接受

Master 审查无误后，可以接受该次合并请求。点击 `Accept Merge Request` 按钮将自动合并分支，勾选 `Remove source-branch` 将同时删除该特性分支。

整个自动合并过程如果以命令形式手工执行的话，步骤如下：

```bash
#Step 1. Update the repo and checkout the branch we are going to merge 
git fetch origin
git checkout -b test origin/feature-test

#Step 2. Merge the branch and push the changes to GitLab 
git checkout release-2016.4.7
git merge --no-ff feature-test
git push origin release-2016.4.7
```

以[非快进式合并](/2015/08/17/git-merge/#非快进式合并)完成后，祖先图谱（graph）的展现结果如下：

```
*   be512fa (HEAD, origin/release-2016.4.7, release-2016.4.7)  Merge branch 'test' into 'release-2016.4.7'
|\
| * 1f52adf 测试
|/
*   a4febbb (tag: 1.0.0, origin/master) 格式化货币保留两位小数
```

最后需要注意的是，只有 `Assignee` 才能够接受合并请求，其它人只会被通知：

> You don't have permission to merge this MR

# 总结

GitLab 提供的上述功能非常实用，为项目的源码管理提供了有力的支持。