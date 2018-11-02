---
title: "Git 分支模型总结"
date: 2016-04-03 22:31:51
updated: 2016-04-03 22:31:51
tags: Git
---

项目总归要协作开发，在此总结我在团队中推广使用的分支模型。

![A successful Git branching model](/img/git/git_branch.png)

# 分支模型

## 主分支（Main branches）

企业的项目开发不像开源的项目开发，通常只会有一个远程仓库。这种情况下，通常会有两个常驻分支：

| Branch Name | Is locked? | Description                              |
| ----------- | ---------- | ---------------------------------------- |
| `master`    | YES        | 主干分支，仅用于发布新版本，平时不能在上面干活，只做代码合并、以及打标记（`git tag`）。<br/> 理论上，每当对 `master` 分支有一个合并提交操作，我们就可以使用 Git 钩子脚本来自动构建并且发布软件到生产服务器。 |
| `dev`       | NO         | 开发分支，平时干活的地方。每当发版时，需要被合并到 `master`。      |

对于简单的项目而言，这样的分支模型已经够用了。

## 辅助性分支（Supporting branches）

除了常驻分支，通常大的特性开发或生产缺陷修复还建议创建相应的临时分支。因为：

1. 在分支上开发可以让你随意尝试，进退自如，比如碰上无法正常工作的特性或补丁，可以先搁在那边，直到有时间仔细核查修复为止。
2. 团队中如果有代码审查流程，独立的分支还可以留给审查者抽空审查的时间和改进代码的余地，并将是否合并、是否发布的权利留给审查者，为代码质量设一道门槛。

每一类分支都有一个特定目的，如何命名每一类分支？建议用相关的主题关键字进行命名，并且建议将分支名称分置于不同**命名空间（前缀）**下，例如：

| Branch Name | May branch off from | Must merge back into | Is locked? | Description                                                  |
| ----------- | ------------------- | -------------------- | ---------- | ------------------------------------------------------------ |
| `feature-*` | `dev`               | `dev`                | NO         | 特性分支，为了开发某种特定功能而建。开发完成并测试通过后，需发送 Merge Request 到 `release-*` 进行代码审查及合版。 |
| `release-*` | `dev`               | `dev` <br/> `master` | YES        | 预发布分支，为了新版本的发布做准备，一般命名为 `release-<版本号>`。这是一个稳定分支，只接受审核通过的 Merge Request。 |
| `hotfix-*`  | `master`            | `dev` <br/> `master` | NO         | 补丁分支，为了修复生产缺陷而建，一般命名为 `hotfix-<issue 编号>` |

与主分支不同，这些辅助性分支总是有一个有限的生命期，因为他们在被合并到主分支之后，就会被移除掉。

# 参考

* 《[Git 分支管理策略 - 阮一峰](http://www.ruanyifeng.com/blog/2012/07/git.html)》
* 《[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)》