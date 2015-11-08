title: "Git 安装与配置"
date: 2015-08-03 11:59:01
updated: 2015-09-12 23:59:01
tags: Git
---

要想 Git 用得爽，首先要安装与配置好。

# 如何安装？

## Windows

需要安装 Git for Windows 的客户端 [msysgit](https://github.com/msysgit/msysgit)。推荐使用 [绿色便携版](http://git-scm.com/download/wint) ，优势如下：

* 无需安装，无需写注册表，无需管理员权限；
* 可以从任意目录运行，甚至 U 盘；

与安装版的区别：

* 不提供右键上下文菜单（如 `Git GUI Here`、`Git Bash Here`），因为该功能需要写入注册表；
* 不修改环境变量 `%path%` ，因此无法在命令行工具 `cmd` 中直接运行 `git.exe` 和 `gitk.exe`，解决办法：
  * 推荐使用自带的 `Git Bash` （类 Unix Shell）或 `Git Cmd` 进行替代；
  * 或将 `%GIT_HOME%\cmd` 目录永久加入环境变量 `%path%` （如果只想在当前会话中临时使用，只需在 `cmd` 中运行 `set path=%GIT_HOME%\cmd;%path%` 即可），然后运行 `git --help` 测试配置效果；

## Linux / Unix

使用包管理 apt-get 或 yum 即可。

## OS X

Homebrew 是最快最便捷的安装方式：

```bash
$ sudo brew install git
```

# 如何配置？

## 配置提交作者

开始使用 Git 之前，第一件重要的事情就是配置提交作者，首先做如下检查：

```bash
$ git config -l | grep user
```

如果返回为空表示未配置，需要配置如下：

```bash
$ git config --global user.name "你的姓名"
$ git config --global user.email "你的邮箱"
```

现在，可以愉快的开始使用 Git 了。更多配置请参考 [这里](http://git-scm.com/docs/git-config) 。

# 常见问题

Git 长期使用上的一点不便，解决方案仅供参考。

## 配置工作目录

Git 默认使用程序运行目录作为工作目录，这会带来使用上的不便。解决办法是新建 `.bashrc` 文件：

```bash
$ vim ~/.bashrc
```

添加一行：

```bash
# 请将 /d/Repos/ 替换成你的仓库目录
cd /d/Repos/
```

即可自动切换到本地仓库所在目录。

## 配置 SSH 代理和密钥

另一个潜在的问题是运行 `Git Bash` 并拉取远程仓库提示错误：

```bash
Could not open a connection to your authentication agent.
```

这是因为 `ssh-agent` 未随 `bash` 一起启动。你可以每次都手工启动，或推荐编写脚本自启动。

新建 `.bashrc` 文件，并添加如下代码：

```bash
#!/bin/sh
if [ -f ~/.agent.env ]; then
  . ~/.agent.env >/dev/null
  kill -15 $SSH_AGENT_PID
fi

echo "Starting ssh-agent..."
eval `ssh-agent |tee ~/.agent.env`
ssh-add ~/.ssh/github_rsa
```

这将会自启动 `ssh-agent` 并添加指定私钥。 `ssh-agent` 是一个密钥管理器，运行 `ssh-agent` 以后，使用 `ssh-add` 将指定私钥交给 `ssh-agent` 保管，其它程序（例如 git）在需要身份认证的时候，可以将认证申请交给 `ssh-agent` 来代为完成整个认证过程。

那么以后每次运行 `Git Bash` 的时候，就会看到输出效果如下：

```bash
Starting ssh-agent...
Agent pid 8828
Identity added: ~/.ssh/github_rsa
```

# 参考

* 《[How To Use Environment Variables in Windows](http://best-windows.vlaurie.com/environment-variables.html)》
* 《[Unix / Linux ssh-add Command Examples to Add SSH Key to Agent](http://linux.101hacks.com/unix/ssh-add/)》
* 《[自定义 Git - 配置 Git](http://git-scm.com/book/zh/v1/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E9%85%8D%E7%BD%AE-Git)》
* 《[Git for Windows 1.x 归档版本下载地址](https://github.com/msysgit/msysgit/tags)》
