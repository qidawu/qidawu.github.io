---
title: Git 实战系列（十四）Git 远程仓库迁移
date: 2017-04-15 23:53:36
updated:
tags: Git
---

# 1.获取权限

首先获取相关 Group 的 Owner 权限。

# 2.在线迁移

![Git 迁移项目](/img/git/git_transfer_project.png)

# 3.更新本地仓库

更新本地仓库，首先查看当前 remote url：

```bash
$ git remote -v
origin  git@git.kd.ssj:finance-ssjmarket/finance-market.git (fetch)
origin  git@git.kd.ssj:finance-ssjmarket/finance-market.git (push)
```

使用 `git remote set-url`  重置 remote url：

```bash
$ git remote set-url origin git@git.kd.ssj:finance-web/finance-market.git
```

检查重置是否成功：

```bash
$ git remote -v
origin  git@git.kd.ssj:finance-web/finance-market.git (fetch)
origin  git@git.kd.ssj:finance-web/finance-market.git (push)
```

# 4 批量更新本地仓库

适用于一堆 git 仓库放在同一个目录下，可以用这个方法进行批量替换：

1. 检查一下现在的 url：

```
cat ./finance-*/.git/config | grep 'git@'
```

2. 批量替换：

```
ls -1 ./仓库名-*/.git/config | xargs  sed -i 's/git@.*\:/git@github.com:/g'
```
3. 再次检查一下结果：

```
cat ./finance-*/.git/config | grep 'git@'
```

