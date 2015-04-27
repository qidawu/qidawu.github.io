title: "Docker 启动失败问题处理"
date: 2015-04-03 13:52:11
updated: 
tags: Docker
---

Q1 参加了一场 Docker 技术分享后，发现了 Docker 这个好东西，回头动手安装，打算跑起来体验一番，却遇到了一些蛋疼的问题。

# 如何安装 Docker

Windows 如何安装 Docker？参考 [官网安装教程](https://docs.docker.com/installation/windows/) 。

由于 Docker 引擎使用了 Linux 特有的内核特性，因此在 Windows 中运行 Docker 需要使用一个轻量级的虚拟机。方便起见，Docker 团队设计了一个名为“Boot2Docker”的辅助程序用于安装所需的虚拟机并用于运行 Docker 进程。

此外，由于 Docker 对镜像（Image）的增删改查操作都是通过 Git ，因此在 Windows 下，还需要一个 MSYS-git 客户端。

因此，Boot2Docker 内置：

* Boot2Docker
* Boot2Docker Management Tool
* [VirtualBox](https://www.virtualbox.org)
* [MSYS-git](http://msysgit.github.io/)

需要注意，如果之前本地已经安装过 Git / GitHub for Windows 、VirtualBox ，则无需再安装内置的 MSYS-git、 VirtualBox 。

# VirtualBox 导致的启动失败问题

## 初始化（docker init）失败问题处理

安装成功后，使用 Git Shell 运行启动脚本，结果却报错：

```
$ start.sh
copying initial boot2docker.iso (run "boot2docker.exe download" to update)
initializing...
error in run: Failed to initialize machine "boot2docker-vm": exit status 1
```

通过阅读 start.sh 脚本，发现卡在了 docker init 初始化这个步骤：

```
echo 'initializing...'
./boot2docker.exe init
```

这时候，通过错误日志或控制台输出排查错误就很关键了，`-v` 参数可以用于显示详细的命令调用过程：

```
-v, --verbose=false: display verbose command invocations.
```

```
$ boot2docker.exe init -v
Boot2Docker-cli version: v1.4.1
Git commit: 43241cb
2015/04/03 11:57:06 executing: E:\Developer\Oracle\VirtualBox\VBoxManage.exe showvminfo boot2docker-vm --machinereadable
2015/04/03 11:57:06 executing: E:\Developer\Oracle\VirtualBox\VBoxManage.exe showvminfo boot2docker-vm --machinereadable
2015/04/03 11:57:06 executing: E:\Developer\Oracle\VirtualBox\VBoxManage.exe list vms
VBoxManage.exe: error: Failed to create the VirtualBox object!
VBoxManage.exe: error: Code E_INVALIDARG (0x80070057) - One or more arguments are invalid (extended info not available)
VBoxManage.exe: error: Most likely, the VirtualBox COM server is not running or failed to start.
error in run: Failed to initialize machine "boot2docker-vm": exit status 1
```

通过控制台输出发现，VirtualBox 下的 `VBoxManage.exe` 运行出错了。

当时很疑惑，本地使用的是 Docker 内置 VirtualBox ，打开 `VirtualBox.exe`，同样会报错。

翻了下 VirtualBox 日志文件，Google 了一番输出错误，貌似跟什么 [Windows 破解主题的 DLL 文件](http://www.dotcoo.com/virtualbox-uxtheme)有关，折腾了一番没能解决。

[再次搜索](http://www.cnblogs.com/tanhangbo/p/4282605.html)，最后发现了两个问题：

1. 程序兼容性问题导致 VirtualBox 无法打开；
2. VirtualBox 版本问题导致虚拟机无法启动；

第一个问题，通过对 `VBoxManage.exe`、`VBoxManage.exe` 右键 - 勾选“以兼容模式运行这个程序”，至少保证打开程序不会报错。

```
$ Oracle/VirtualBox/VBoxManage.exe list vms
"centos" {440faef3-b83d-4daa-b7ce-3942c03dc580}
```

第二个问题，通过多个版本的安装/卸载尝试，最后发现在我的电脑使用 VirtualBox 4.3.10 能够正常启动虚拟机。

## 启动（docker start）失败问题处理

Docker 成功初始化后，还遇到一个虚拟化失败导致的 VirtualBox 无法启动的问题，报错如下：

```
......
VMSetError: VT-x is disabled in the BIOS.
......
```

打开 BIOS 的 Virtualization Technology (VTx/VTd) 开关，即可解决问题。

# 最后

再次运行 Docker 启动脚本，Docker 终于启动起来了！

```
connecting...
                        ##        .
                  ## ## ##       ==
               ## ## ## ##      ===
           /""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
           \______ o          __/
             \    \        __/
              \____\______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
```

# 参考

《[virtualbox 打不开ubuntu解决](http://www.cnblogs.com/tanhangbo/p/4282605.html)》