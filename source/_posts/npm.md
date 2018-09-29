---
title: npm 常用命令及配置总结
date: 2018-09-29 19:15:54
updated:
tags: 前端
---

工作中由于前端项目打包、各类工具使用（如本博客就是基于 Node.js）经常要用到 npm，下面总结一下。

npm 是随 Node.js 自动安装的，由三个独立的部分组成：

* [网址](https://npmjs.com/)：开发者查找包（package）、设置参数以及管理 npm 使用体验的主要途径。
* [注册表（Registry）](https://docs.npmjs.com/misc/registry)：一个巨大的数据库，保存了每个包（package）的信息。
* [命令行工具（CLI）](https://docs.npmjs.com/cli/npm)：命令行或终端运行。开发者通过 CLI 与 npm 打交道。

# CLI 命令

![NPM CLI Commands](/img/javascript/npm_cli_commands.png)

# npm 配置

npm 配置主要有两份：

* [npmrc](https://www.npmjs.com.cn/files/npmrc/)：npm 从命令行、环境变量、`npmrc` 文件中读取配置。
* [package.json](https://www.npmjs.com.cn/files/package.json/)：包（package）配置文件。

重点了解下 package.json，重点属性有 `name`、`version`、`dependencies`、`main`：

| 属性           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| `name`         | 包名                                                         |
| `version`      | 包的语义版本号：X.Y.Z：如果有大变动，向下不兼容，需要更新主版本号X；如果是新增了功能，但是向下兼容，需要更新次版本号 Y；如果只是修复bug，需要更新补丁版本号 Z |
| `description`  | 包的描述                                                     |
| `homepage`     | 包的官网 url                                                 |
| `author`       | 包的作者姓名                                                 |
| `contributors` | 包的其他贡献者姓名                                           |
| `dependencies` | 依赖包列表。如果依赖包没有安装，npm 会自动将依赖包安装在 node_module 目录下 |
| `repository`   | 包代码存放的地方的类型，可以是 git 或 svn，git 可在 Github 上 |
| `main`         | `main` 字段是一个模块ID，它是一个指向你程序的主要项目。就是说，如果你包的名字叫 express，然后用户安装它，然后`require("express")` |
| `keywords`     | 关键字                                                       |

# 参考

https://www.npmjs.com/

https://www.npmjs.com.cn/

https://npm.taobao.org/

