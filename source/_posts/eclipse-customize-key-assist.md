---
title: "Eclipse 系列之自定义高效快捷键"
date: 2015-04-19 14:45:44
updated: 
tags: Java
description: "好的快捷键可以提高编码的效率。"
---

> 好的快捷键可以提高编码的效率。

# 第三方解决方案

曾经使用过 Eclipse 的 Vim 插件 [viplugin](http://www.viplugin.com/) 提升光标移动和文本编辑的效率，但终因一些热键冲突问题而放弃使用了，有兴趣的同学可以参考 《[viplugin 用户手册](http://viplugin.com/files/User_Manual_viPlugin.pdf)》，付费破解参考[这里](http://blog.163.com/thomaskjh@126/blog/static/37082998201231810718341/)。

其它解决方案还有 [vrapper](http://vrapper.sourceforge.net/)、[eclim](http://eclim.org/) ，可以自行体验。

# 我的方案

经过权衡，还是决定自定义一套属于自己的快捷键。键位主要参考 Vim 和 Bash ，并主要利用闲置的 `Alt` 键避免冲突，配置总结如下：

## 移动光标类

| Command               | Binding           | Description |
| --------------------- | ----------------- | ----------- |
| Line Up               | `Alt + K`         | ↑           |
| Line Down             | `Alt + J`         | ↓           |
| Previous Column       | `Alt + H`         | ←           |
| Next Column           | `Alt + L`         | →           |
| Line Start            | `Alt + A`         |             |
| Line End              | `Alt + E`         |             |
| Previous Word         | `Alt + B`         |             |
| Next Word             | `Alt + F`         |             |
| Go to Next Member     | `Alt + G`         |             |
| Go to Previous Member | `Alt + Shift + G` |             |
| Open Implementation   | `Alt + Q`         |             |

## 通用类

| Command                  | Binding              |
| ------------------------ | -------------------- |
| Delete                   | `Alt + D`            |
| Select Enclosing Element | `Alt + S`            |
| File Search              | `Ctrl + H`           |
| Next View                | `Alt + V`            |
| Rerun JUnit Test         | `Alt + R`            |
| Run Maven Clean          | `Alt + Shift + X, C` |

## Git 类

| Command                    | Binding   |
| -------------------------- | --------- |
| Pull                       | `Alt + 1` |
| Commit                     | `Alt + 2` |
| Show in History            | `Alt + 3` |
| Compare with HEAD Revision | `Alt + 4` |
| Replace with HEAD Revision | `Alt + 5` |
| Show In (Git Repositories) | `Alt + 6` |

## SVN 类

| Command                          | Binding   |
| -------------------------------- | --------- |
| 更新                               | `Alt + 1` |
| 提交                               | `Alt + 2` |
| Show History                     | `Alt + 3` |
| Compare with Local Base Revision | `Alt + 4` |
| 复原                               | `Alt + 5` |

注意，Eclipse 有 Bug 在配置完以上 SVN 快捷键之后，快捷键是无法使用的，需要为当前的视图（Perspective）添加一个 SVN 的工具。配置如下：

> Window > Customize Perspective... > Tool Bar Visibility

勾选 SVN 即可。

# 另辟蹊径的方案

其实也可以不必局限于 Eclipse 这款 IDE ，热爱 Vim 快捷键的同学可以考虑使用其它 IDE ，如 [Sublime Text](http://www.sublimetext.com/) ，可以原生完美支持 Vim 快捷键，配置如下：

> Preferences - Settings - User - "ignored_packages": []

去掉 [] 里面的 “Vintage” 即可。

在写 Python 代码时就感觉体验很好，值得一试。