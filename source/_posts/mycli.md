---
title: MyCli：支持自动补全和语法高亮的 MySQL 客户端
date: 2017-03-02 00:05:01
updated:
tags: MySQL
---

工作中常用到 mysql 自带的命令行工具，但实在难用。推荐一款 MySQL 命令行工具——[MyCli](http://mycli.net/)，支持**自动补全**和**语法高亮**。也可用于 MariaDB 和 Percona。

功能如下：

![MyCLI](/img/mysql/mycli.gif)

MyCLI 的兼容性爆表，支持 Windows、MacOS、Linux，运行在 Python 2.7, 3.3, 3.4, 3.5, 3.6。安装简易，例如 Windows 只要安装了 [Python 环境](https://www.python.org/downloads/)及其包管理工具 `pip` ，就能一键安装：

```bash
$ pip install mycli

or

$ easy_install mycli
```

其它系统的安装方式，请参考：http://mycli.net/install

Windows 下使用 `cmd` 连接数据库，如下：

```
mycli -hlocalhost -P3306 -uroot -p123456
```

