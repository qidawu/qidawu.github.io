---
title: Hexo 博客搭建 & 自动化构建、部署
date: 2019-05-26 21:56:00
updated:
tags: 建站
typora-root-url: ..
---

Hexo 博客使用好多年了，总结下日常使用的一些内容。

# Hexo 博客搭建

## 安装配置

```bash
# -g 参数全局安装 Hexo 命令行工具，安装后才可以使用下述 hexo 命令
$ npm install -g hexo-cli

# 初始化本地仓库及 hexo 文件，适用于第一次使用
$ hexo init

# hexo 基础配置、主题、插件配置等等，详细配置参考官网
$ vim _config.yml

# 根据 package.json 的声明（hexo 版本及 dependencies 版本）安装所需依赖到当前目录 node_modules
$ npm install
```

## 主题

NexT

https://github.com/next-theme/hexo-theme-next

https://theme-next.js.org/pisces/

https://hexo-next.readthedocs.io/zh_CN/latest/

## 常用命令

依赖安装完毕，开始使用 hexo，常用命令如下：

![Hexo 常用命令](/img/hexo/hexo_cmd.png)

# 自动化构建 & 部署

推荐使用 GitHub Actions，简单、免费，而 Travis CI 收费了。

## GitHub Actions

为了方便随时随地可以编写博客，搭建好的本地仓库及其源文件一般会推送到 GitHub 远程仓库中保管，并自动构建 & 部署到 GitHub Pages 服务。步骤如下：

1. 创建 GitHub Pages 服务所需的仓库 `yourname.github.io`，注意该仓库必须是 public 权限。
2. 创建 `.github/workflows/pages.yml` 配置文件。
3. 推送源文件至该仓库。

参考：https://hexo.io/docs/github-pages

## Travis CI

完成上面两步就可以开始创作了。但毕竟命令还是有些繁琐，因此可以利用持续集成服务代替人工来做重复的事情。引入 Travis CI 后，整体流程如下：

![GitHub Pages with CI](/img/hexo/github-pages-ci.png)

从上述流程来看，作者只需要完成创作并推送即可，其它构建、部署的事则由 Travis CI 来完成，非常简单。

下面来看下如何配置：

### GitHub 创建 access token

登录 GitHub - Settings - Developer Settings 选项，找到 Personal access tokens 页面，创建个人 access token，创建时权限 `repo` 权限和 `user:email` 权限。

### Travis CI 仓库配置

1. 使用 GitHub 账户登录 [Travis CI](https://travis-ci.org/) 官网并进行 OAuth 授权
2. 同步仓库一，过程中会在 GitHub 账户下安装 Travis CI 的 GitHub App，用于触发持续集成
3. 为仓库一设置环境变量：
   - `GH_TOKEN` 值为 GitHub access token
   - `GH_REF` 值为 GitHub 仓库二地址

### 创建 .travis.yml 配置

```YML
# 设置语言
language: node_js
# 设置相应的版本，可以指定版本 10，或者使用稳定版 stable
node_js: stable
# 设置只监听哪个分支
branches:
  only:
    - master
# 缓存 node_modules 目录，可以节省持续集成的时间
cache:
  directories:
    - node_modules
before_install:
  - npm install -g hexo-cli
install:
  - npm install
script:
  - hexo clean
  - hexo g
after_script:
  - cd ./public
  - git init
  - git config user.name "yourname" # 修改name
  - git config user.email "youremail" # 修改email
  - git add .
  - git commit -m "Travis CI Auto Builder"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master
```

### 自动构建

创作并推送到仓库一即可，其它构建、部署的事都由 Travis CI 来完成，过程如下：

![Travis CI](/img/hexo/travis_ci.png)