---
title: git diff 命令对比文件差异
date: 2016-04-01 22:27:23
updated:
tags: Git
---

对比两个分支中，所有文件的详细差异，常用于合并操作之后确认有没有遗漏文件：

```bash
$ git diff branch1 branch2
```

对比两个分支中，指定文件的详细差异：

```bash
$ git diff branch1 branch2 文件名(带路径)
```

对比两个分支中，差异的文件列表：

```bash
$ git diff branch1 branch2 --stat
```

