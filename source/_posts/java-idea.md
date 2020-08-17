---
title: IDEA 一点小总结
date: 2018-07-29 12:01:58
updated:
tags: Java
typora-root-url: ..
---

年初也转投 IDEA 阵营了，相比 Eclipse 确实顺畅很多，尤其是多开项目的时候，已经有种回不去的感觉了。这里介绍一些两个 IDE 之间的区别。

# 版本选择

首先是 IDEA 版本的选择，建议使用 Community 社区免费版即可，对比 Ultimate 收费版并无太大区别。都支持 Java、Maven、Git 等基本功能。但社区版会少了一些插件，例如 Spring Initializr。

版本区别详见：https://www.jetbrains.com/idea/features/editions_comparison_matrix.html

# 创建项目

其次创建项目，需要了解 IDEA 的 Project 和 Module 两个概念：

- IntelliJ 系中的 Module 相当于 Eclipse 系中的 Project；
- IntelliJ 系中的 Project 相当于 Eclipse 系中的 Workspace。可以选择创建一个 Empty project without modules，之后再放入同一类型的 module 集合，此时只会生成一个 .idea 目录。也可以选择直接创建某一类型的 Project，如 Maven Project，此时需要添加 Maven 项目信息，完成后会生成 pom.xml、src 目录、*.iml。

最后尝试新建一个 HelloWorld 示例项目，`.idea` 文件夹和 `HelloWorld.iml` 是 IDEA 帮助我们建立的辅助文件夹和文件，类似于 Eclipse 在我们的 Workspace 下建立的 `.settings` 文件夹和 `.classpath` 、`.project` 文件。 

![IDEA Hello World](/img/java/idea/hello_world.png)

Project 作为工作空间可以自由切换、多开，从 File > Open Recent 中选择即可。

# 配置 Tomcat

1、下载 Tomcat

2、IDEA 新建 Tomcat（Run > Edit Configurations）

![add_new_tomcat_server_config](/img/java/idea/add_new_tomcat_server_config.png)

3、新建完毕，配置本地 Tomcat 安装目录：

![tomcat_server_config](/img/java/idea/tomcat_server_config.png)

4、配置端口号、必要的 VM options：

![tomcat_server_config2](/img/java/idea/tomcat_server_config2.png)

5、配置运行项目，配置上下文：

![deployment](/img/java/idea/deployment.png)

# 创建 java/resource 目录

`Ctrl+Alt+Shift+S`，打开 Project Structure > Modules：

![Project Structure > Modules](/img/java/idea/project_structure_modules.png)

# 场景演示

以特性开发与合版这个场景为例，看下如何利用 IDEA 完成整个流程：

## 特性开发

第一步，本地建一个工作目录，以特性名称命名：

```bash
$ mkdir feature-xxx
```

第二步，从远程仓库 `clone` 该特性涉及的所有模块到该工作目录：

```bash
$ cd !$
$ git clone module1
$ git clone moduleN
```

第三步，新建 Project。打开 IDEA，File > New > Project from Existing Sources... > 选择该工作目录，并按如下操作：

![Import Module](/img/java/idea/import_module.png)

![Import Module](/img/java/idea/import_module_search_for_projects_recursively.png)

第四步，开始特性开发，批量从 master 分支 `checkout` 创建出特性分支 `feature-xxx`。并批量修改 `pom.xml` 的版本号：

![new_branch](/img/java/idea/new_branch.png)

## 特性合版

第一步，项目管理员创建发布分支，根据本次发布周期涉及的所有特性，批量从 master 分支 `checkout` 创建出发布分支 `release-xxx`（操作同上）。该操作只能由项目管理员操作，因为 `release-*` 是受保护分支。

第二步，升级 `pom.xml` 版本号并推送。

第三步，批量 `merge` 特性分支：

![merge_into_current](/img/java/idea/merge_into_current.png)

第四步，解决冲突（例如 `pom.xml` 版本号冲突、代码冲突，如有）：

![conflicts](/img/java/idea/conflicts.png)

![version_control_local_changes](/img/java/idea/version_control_local_changes.png)

第五步，合版完毕，`Ctrl+K` 批量提交并推送到远程分支 `release-xxx` 即可。

# 全局配置

File > Other Setting > Default Settings

全局配置 IDE 的界面、编辑器、VCS、构建、终端等

File > Other Setting > Default Project Structure

全局配置项目的 SDK 版本、类库等。

## JDK

File > Other Setting > Default Settings，`Projects SDK` 可控制全局版本，`Language Level` 可控制语言级别，方便使用其特性。配置后全部 project modules 都会生效。

## Maven

File > Other Setting > Default Settings > Build, Execution, Deployment > Build Tools > Maven

* 可配置 `Maven home directory`、`User settings file` (自定义 setting.xml)、`Local repository` 本地仓库。

* 自动下载源码：Importing，找到`Automatically download` 并勾选 `Sources` 和 `Documentation`，然后 reimport 即可。

## Terminal

File > Other Setting > Default Settings > Terminal，修改 `Shell path` 为：`D:\Developer\PortableGit\bin\sh.exe`，可配置 Terminal 终端为 GitBash。

## Layout

调整完当前窗口的布局之后，可以保存当前的窗口布局，以便全局生效：Window > Store Current Layout as Default

## Code Template

File > Other Setting > Default Settings > 搜索 File and Code Templates，打开 Includes > File Header，配置：

```
/**
 * <p>
 *
 * </p>
 *
 * @author abc@xyz.com
 * @since ${DATE}
 **/
```

## Auto Import

File > Settings > Editor > General > Auto Import，然后勾选：

`Add unambiguous imports on the fly`：快速添加明确的导入。

`Optimize imports on the fly`：快速优化导入，优化的意思即自动帮助删除无用的导入。

## Quick documentation 

File > Settings > Editor > General > 开启 `Show quick documentation on mouse move`，可用于鼠标放到类、方法、变量上时显示完整 java doc 注释

## Show tabs in one row

File > Settings > Editor > General > Editor Tabs，去掉勾选 `Show tabs in one row`

# 字体

老版本可以[安装 JetBrains Mono 字体](https://www.jetbrains.com/lp/mono/#how-to-install)，v2019.3 版本后自带。

# 插件

官方插件（内置）：

- Maven Integration：官方 Maven 插件，参考：[IDEA Maven 插件](/2018/05/01/maven-build-lifecycle/)
- JUnit：快速创建、运行、查看单元测试，在单元测试和目标之间跳转。

官方插件（需安装）：

- IdeaVim：Vim 程序员必装。
- NodeJS：运行前端项目、or 构建前端项目时都是需要的。

规范类：

- Alibaba Java Coding Guidelines：阿里巴巴 Java 开发规范
- Git Commit Template：参考
  - [Git 代码提交规范](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
  - [如何规范你的Git commit？](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247498589&idx=1&sn=c0419f08bd455de9147e47387778943e)

代码生成类：

- Lombok Plugin：用于精简冗余代码，必装。
- [Builder-Generator](https://plugins.jetbrains.com/plugin/6585-builder-generator)：如果项目可以使用 Lombok，就不必装。

MyBatis：

- ~~[MyBatis Plugin](https://plugins.jetbrains.com/plugin/7293-mybatis-plugin)：最强大的 MyBatis 插件，不过是收费版。~~
- Free MyBatis plugin：免费的 MyBatis 插件，提供了一些基本的跳转和代码生成功能。
- MyBatis Log Plugin：把 MyBatis 输出的 SQL 日志还原成完整的 SQL 语句。将日志输出的 SQL 语句中的问号 ? 替换成真正的参数值。

辅助类：

* Maven Helper：方便查询依赖树及排除依赖
* Grep Console：Grep, tail, filter, highlight... everything you need for a console. Also can highlight the editor - nice for analyzing logs...
* SequenceDiagram for IntelliJ IDEA：查看类调用时序图
* Rainbow Brackets：彩色括号
* HighlightBracketPair：高亮提示括号的开始结尾

其它：

- [Material Theme UI](https://plugins.jetbrains.com/plugin/8006-material-theme-ui)：将默认外观改为 [Material Design](https://material.io/) 风格。
- [Smart Tomcat](https://plugins.jetbrains.com/plugin/9492-smart-tomcat)：社区版没有提供该功能（[点我](https://stackoverflow.com/questions/22047860/tomcat-in-intellij-idea-community-edition)），可以用这个插件替代
- [Ace Jump](http://kidneyball.iteye.com/blog/1814028)：使用这个插件，直接使用键盘定位到你想去的地方 。

IdeaVim 如果与原 IDEA 快捷键冲突，可以修改如下：

![IdeaVim](/img/java/idea/IdeaVim.png)

如果被墙导致无法连接到 Marketplace 或插件无法下载，可以配置如下：

![IDEA 无法下载插件](/img/java/idea/use_secure_connection.png)

# 快捷键

## 自定义 keymap

自定义快捷键，例如我配置了：

| 功能                                     | 快捷键       |
| ---------------------------------------- | ------------ |
| Close                                    | `Alt+W`      |
| Hide All Tool Windows                    | `Alt+M`      |
| Delete Line                              | `Alt+D`      |
| Redo                                     | `Ctrl+Y`     |
| Rename                                   | `Alt+R`      |
| Completion                               | `Alt+/`      |

## VCS

Git 自定义快捷键：

| 功能                                     | 快捷键       |
| ---------------------------------------- | ------------ |
| Reset HEAD                               | `Ctrl+Alt+Q` |
| Annotate（git blame）                    | `Ctrl+Alt+1` |
| Compare with the Same Repository Version | `Ctrl+Alt+2` |
| Show History                             | `Ctrl+Alt+3` |
| Commit File                              | `Ctrl+Alt+4` |
| PULL                                     | `Ctrl+Alt+5` |

Git 默认的其它命令：

| 功能                            | 快捷键         |
| ------------------------------- | -------------- |
| Add                             | `Ctrl+Alt+A`   |
| Revert Changes 回退更改         | `Ctrl+Alt+Z`   |
| Commit Changes 提交更改         | `Ctrl+K`       |
| Push Commits 推送               | `Ctrl+Shift+K` |
| Update Project（批量 git pull） | `Ctrl+T`       |

## 创建类

> Alt + Insert 创造万物

## 查找/替换类

> Ctrl + F 在当前文件中查找（关键字）
>
> Ctrl + R 在当前文件中替换（关键字）
>
> Ctrl + Shift + F 在路径中查找（关键字）
>
> Ctrl + Shift + R 在路径中替换（关键字）

## 浏览类

> Ctrl+N 查找类
>
> Ctrl+Shift+N 查找文件
>
> Shift+Shift 查找所有（包括类、资源、配置项、方法等等）



> Ctrl+E 最近打开的文件
>
> Alt+F7 查找引用（Find Usage）
>
> Ctrl+Shift+Backspace 回到上次修改的地方
>
> Ctrl+Alt+Left/Right 返回/前进至上次浏览的位置
>
> Ctrl+鼠标左键：打开接口（Declaration）
>
> Ctrl+Alt+鼠标左键：打开实现（Implementation(s)）



> Ctrl+H 打开类层次窗口（继承关系）
>
> Ctrl+F12 查看当前类的所有方法
>
> Ctrl+Alt+U 类关系图、Maven关系图

## 编辑类

> Ctrl+D 复制行
>
> Ctrl+Y 删除行
>
> Alt+Shift+向上箭头 Move Line Up
>
> Alt+Shift+向下箭头 Move Line Down
>
> Ctrl+Shift+U 大小写转换
>
> Ctrl+Alt+L 格式化代码
>
> Ctrl+W、Ctrl+Shift+W 这个动作的实际操作是选中更上一层的语法结构。例如，如果你在一个字符串的一个单词中，按一下Ctrl+W，会选中光标所在单词。再按一下，会选中整个字符串的内容，不包括引号。再按一下，会选中包括引号的字符串。再按一下，会选中整个表达式（如果表达式含有括号，会逐层选中）。再按一下，会选中整个语句块。再按一下，会选中整个方法。再按一下，会选中整个类。

## 重构类

> // Ctrl+Alt+Shift+T 重构菜单
>
> // Ctrl+f6 修改类\方法签名
> https://www.jetbrains.com/help/idea/change-class-signature.html
>
>
> // F6 Move 类、方法、变量移动
>
> // Ctrl+Shift+f6 重构变量的类型
>
> // Shift+f6 重命类、方法、变量
>
> // Ctrl+Alt+C 提取常量
> // Ctrl+Alt+V 提取变量
> // Ctrl+Alt+F 提取成员变量
> // Ctrl+Alt+P 提取方法入参
>
> // Refactor | Generify 泛型自动补全
> // Refactor | Invert Boolean 布尔类型取反
> // Refactor | Make Static 将类、方法设置成静态
> // Refactor | Find and Replace Code Duplicates
>
> // Safe Delete 安全删除
>
> // Ctrl+Alt+M 方法提取
> // Ctrl+alt+N 方法内联(变量\方法\构造函数)
> https://www.jetbrains.com/help/idea/inline.html
>
> // Pull Members Up 将方法提升到父类
> // Pull Members Down 将方法推迟到之类

## 调试类

> 调试：F7/F8/F9 分别对应 Step into，Step over，Continue。
>
> 运行：Alt+Shift+F10 运行程序，Shift+F9 启动调试，Ctrl+F2 停止。

## 总结

![IDEA 快捷键](/img/java/idea/keymap.png)

摘录 10 个最常用的快捷键：

> Top #10切来切去：Ctrl+Tab
>
> Top #9选你所想：Ctrl+W
>
> Top #8代码生成：Template/Postfix +Tab
>
> Top #7发号施令：Ctrl+Shift+A
>
> Top #6无处藏身：Shift+Shift
>
> Top #5自动完成：Ctrl+Shift+Enter
>
> Top #4创造万物：Alt+Insert
>
> Top #3智能补全：Ctrl+Shift+Space
>
> Top #2自我修复：Alt+Enter
>
> Top #1重构一切：Ctrl+Shift+Alt+T

# 参考

下载地址：
http://www.jetbrains.com/idea/download/#section=windows

插件：
https://plugins.jetbrains.com/idea

[常用插件推荐](<https://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247488006&idx=1&sn=d5c66d84724b1deebac6604749d04bf5&chksm=ebd62d2adca1a43cb136b5740621e25854537054b9b3cac7451fd21ea55c0fc247e07a49d8cd&mpshare=1&scene=23&srcid=#rd>)

IDEA 重构官方文档：
https://www.jetbrains.com/help/idea/refactoring-source-code.html

IDEA 高级调试技巧：

http://www.mamicode.com/info-detail-1761996.html