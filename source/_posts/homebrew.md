---
title: Homebrew 包管理器总结
date: 2024-02-01 17:48:49
updated:
tags: 操作系统
typora-root-url: ..
---

参考文档：https://brew.sh/

安装目录：`/usr/local/Cellar`

# Homebrew

![terminology](/img/os/homebrew/homebrew_terminology.png)

- Cask 通常用于安装一些带有图形界面的应用程序。
- Formulae 则是用于安装命令行工具和软件包的规则和脚本。

## cask

常用的 cask：

```bash
$ brew install --cask iterm2
$ brew install --cask windterm
$ brew install --cask redisinsight
$ brew install --cask oss-browser
$ brew install --cask sourcetree

$ brew install --cask typora
$ brew install --cask xmind
$ brew install --cask evernote
```

## formula

常用的 formula：

```bash
$ brew install coreutils

$ brew install node

$ brew install nginx
$ brew install mysql
$ brew install redis
$ brew install elasticsearch
$ brew install zookeeper
$ brew install prometheus
$ brew install grafana

$ brew install ffmpeg
$ brew install imagemagick

$ brew install wireshark
```

## tap

![tap](/img/os/homebrew/homebrew_tap.png)

### MongoDB

https://www.mongodb.com/zh-cn/docs/manual/tutorial/install-mongodb-on-os-x/

https://github.com/mongodb/homebrew-brew

```bash
$ brew tap mongodb/brew
```

## services

![services](/img/os/homebrew/homebrew_services.png)

### 例子

```bash
$ brew services start mysql@8.4
$ brew services start redis
$ brew services start mongodb-community
$ brew services start grafana
```

# 国内加速

《[Homebrew 国内如何自动安装？](https://zhuanlan.zhihu.com/p/111014448)》

Homebrew 源替换为 USTC 镜像：http://mirrors.ustc.edu.cn/help/brew.git.html

Homebrew 关闭自动更新：https://github.com/Homebrew/homebrew-autoupdate

```bash
export HOMEBREW_NO_AUTO_UPDATE="1"
```

# 参考

