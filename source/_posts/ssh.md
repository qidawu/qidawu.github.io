---
title: SSH 介绍与使用
date: 2016-10-06 17:42:15
updated:
tags: GNU/Linux
---

# 连接协议对比

常见的几种连接协议：

* SSH2（默认，相对于SSH1进行了加密算法的改进，使用最广泛）
* SSH1
* Telnet
* Telnet/SSL
* Rlogin
* Serial
* TAPI

在出现 SSH 之前，系统管理员需要登入远程服务器执行系统管理任务时，都是用 `telnet` 来实现的，telnet 协议采用明文密码传送，在传送过程中对数据也不加密，很容易被不怀好意的人在网络上监听到密码。

同样，在 SSH 工具出现之前 R 系列命令也很流行（由于这些命令都以字母 r 开头，故把这些命令合称为 R 系列命令，R 是 remote 的意思），比如 `rexec` 是用来执行远程服务器上的命令的，和 `telnet` 的区别是 `telnet` 需要先登录远程服务器再实行相关的命令，而 R 系列命令可以把登录和执行命令并登出系统的操作整合在一起。这样就不需要为在远程服务器上执行一个命令而特地登录服务器了。

SSH 全称 Secure SHell，顾名思义就是非常安全的 shell 的意思，SSH 协议是 IETF（Internet Engineering Task Force） 的 Network Working Group 所制定的一种协议。SSH 的主要目的是用来**取代传统的 telnet 和 R 系列命令**（`rlogin`、`rsh`、`rexec` 等）远程登录和远程执行命令的工具，实现对远程登录和远程执行命令加密。防止由于网络监听而出现的密码泄漏，对系统构成威胁。

SSH 是一种加密协议，不仅在登录过程中对密码进行加密传送，而且对登录后执行的命令的数据也进行加密，这样即使别人在网络上监听并截获了你的数据包，他也看不到其中的内容。SSH 协议底层使用 TCP 协议，端口号 22。

# OpenSSH

[OpenSSH](http://www.openssh.com/) 是 SSH 协议的**免费开源实现**。OpenSSH 套件使用 `ssh` 程序替代 `telnet` 和 `rlogin` ，使用 `scp` 替代 `rcp` ，使用 `sftp` 替代 `ftp` 。OpenSSH 套件还包含一个 `sshd` 服务端程序（一个运行于服务端的独立守护进程（standalone daemon）），以及一系列 SSH 工具：

* `ssh-add` Tool which adds private keys to the authentication agent.
* `ssh-agent` An authentication agent that can store private keys.
* `ssh-keygen` Key generation tool.
* `ssh-keysign` Helper program for host-based authentication.
* `ssh-keyscan` Utility for gathering public host keys from a number of hosts.

## 鉴权方式

不同于 `telnet` 只支持 Password 密码鉴权，SSH 同时支持以下几种鉴权方式（Authentication）：

* Password（密码）
* **Public Key**（公钥）
* Keyboard Interactive（键盘交互）
* GSSAPI 

目前 SSH 最常用的鉴权方式有 Password 和 Public key 。

###  Public Key

Public Key 非对称（asymmetric）鉴权认证使用一对相关联的 Key Pair（一个公钥 Public Key，一个私钥 Private Key）来代替传统的密码（Password）。顾名思义，Public Key 是用来公开的，可以将其放到 SSH 服务器自己的帐号中，而 Private Key 只能由自己保管，用来证明自己身份。

使用 Public Key 加密过的数据只有用与之相对应的 Private Key 才能解密。这样在鉴权的过程中，Public Key 拥有者便可以通过 Public Key 加密一些东西发送给对应的 Private Key 拥有者，如果在通信的双方都拥有对方的 Public Key（自己的 Private Key 只由自己保管），那么就可以通过这对 Key Pair 来安全地交换信息，从而实现相互鉴权。

## 相关文件

SSH 相关文件和配置：

| 文件                     | 描述                                       |
| ---------------------- | ---------------------------------------- |
| ~/.ssh/id_rsa.pub      | 公钥（Public Key）                           |
| ~/.ssh/id_rsa          | 私钥（Private Key）                          |
| ~/.ssh/known_hosts     | 位于客户端的公钥列表文件，首次与目标主机建立 SSH 连接时，需要添加对方的公钥到这个文件以便后续通信 |
| ~/.ssh/authorized_keys | 位于服务端的公钥列表文件，列出了所有被允许登录进来的可信公钥信息（Lists the public keys that are permitted for logging in） |
| ~/.ssh/config          | SSH 客户端配置文件，可以通过 `man ssh_config` 命令查看帮助。 |

# SSH 技巧

当管理的服务器较多时，ssh 远程需要频繁的输入用户名、密码、服务器 IP，操作非常繁琐，下面介绍两个技巧简化操作。

## 免密登录

在使用 `ssh` 进行远程登录时，由于默认使用的是 Password 鉴权方式，因此每次登录都需要输入密码，操作麻烦。下面介绍 Public Key  鉴权方式进行免密登录。

一、创建一对公私钥：

```bash
$ ssh-keygen -t rsa -C who@where
询问密码时，保持为空并回车
```

二、启动 SSH 认证代理程序：

```bash
$ eval `ssh-agent`
Agent pid 1760
```

三、添加私钥：

```bash
$ ssh-add ~/.ssh/id_rsa
Identity added: ~/.ssh/id_rsa
$ ssh-add -l
2048 8a:63:12:ae:b1:4c:be:03:e7:7f:92:3e:e5:44:56:bb ~/.ssh/id_rsa (RSA)
```

四、将公钥上传到服务端，添加到**被登录帐户**的**可信列表文件**：

```bash
$ scp ~/.ssh/id_rsa.pub who@where:~/.ssh/authorized_keys
```

五、修改服务端文件权限：

```bash
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/*
```

之后再使用 `ssh` 登录时，客户端的 `ssh-agent` 会发送私钥去和服务端上的公钥做匹配，如果匹配成功就可以免密登录了。

##  别名设置

ssh 远程时，通常命令如下：

```bash
$ ssh [-p port] [user@]hostname [command]
```

参数繁琐，一旦服务器较多，要一个个记住并且敲入时非常低效。

ssh 提供了配置文件的方式简化命令行选项。ssh 依序从下列来源中获取配置，最先获取的值将优先使用：

1. 命令行选项（command-line options）
2. 用户的配置文件：`~/.ssh/config`
3. 系统的配置文件：`/etc/ssh/ssh_config`


常用配置项如下：

```bash
Host    别名
    HostName        主机名
    Port            端口
    User            用户名
    IdentityFile    密钥文件的路径
```

通过配置，ssh 远程命令简化如下：

```bash
$ ssh 别名
```

# 常用命令

## ssh

命令用法：

```bash
$ ssh [-p port] [user@]hostname [command]
```

`ssh` 命令执行时，会从下列来源中按序获取配置参数：

1. 命令行参数
2. 用户配置文件 `~/.ssh/config`
3. 系统配置文件 `/etc/ssh/ssh_config`

最先获取到的参数值将被优先使用。



例如：

```bash
$ ssh pc2 /sbin/ifconfig
```

`pc2` 是从 `~/.ssh/config` 中获取的 hostname 别名。

## ssh-add

命令常见用法：

```bash
$ ssh-add -D
All identities removed.

$ ssh-add -l
The agent has no identities.

$ ssh-add -h
ssh-add: unknown option -- h
usage: ssh-add [options] [file ...]
Options:
  -l          List fingerprints of all identities.
  -E hash     Specify hash algorithm used for fingerprints.
  -L          List public key parameters of all identities.
  -k          Load only keys and not certificates.
  -c          Require confirmation to sign using identities
  -t life     Set lifetime (in seconds) when adding identities.
  -d          Delete identity.
  -D          Delete all identities.
  -x          Lock agent.
  -X          Unlock agent.
  -s pkcs11   Add keys from PKCS#11 provider.
  -e pkcs11   Remove keys provided by PKCS#11 provider.
```

参考：http://linux.101hacks.com/unix/ssh-add/

## scp

命令常用参数：

```bash
-r 递归复制（用以传输文件夹）
-p 传输时保留文件权限及时间戳
-C 传输时进行数据压缩
```

### 批量 scp

利用 bash 的 `for` 循环实现批量 `scp` 目录：

```bash
#!/bin/bash

HOST_IP=('192.168.0.1' '192.168.0.2' '192.168.0.3')

for ip in ${HOST_IP[@]}  
  do
    scp -rp /some/files ${ip}:/some/
  done
```

## rsync

批量 `scp` 的缺点是会全量同步，且删除行为无法同步，可以用 `rsync` 命令优化：

```bash
拉取：
$ rsync [option...] [user@]host:src... [dest]

推送：
$ rsync [option...] src... [user@]host:dest
```

### 批量 rsync

```bash
#!/bin/bash

HOST_IP=('192.168.0.1' '192.168.0.2' '192.168.0.3')

for ip in ${HOST_IP[@]}
  do
    rsync -avH --delete /some/* ${ip}:/some/
  done
```

进行如下文件操作测试：

* 新增文件：web-banner-20170717.jpg
* 删除文件：web-banner-20170716.jpg

从输出可见，只会增量同步并删除指定的文件：

```
sending incremental file list
resmarket/static/site/v1/img/banner/
resmarket/static/site/v1/img/banner/web-banner-20170717.jpg
deleting resmarket/static/site/v1/img/banner/web-banner-20170716.jpg
```

# 参考

《[5 Unix / Linux ssh-add Command Examples to Add SSH Key to Agent](http://linux.101hacks.com/unix/ssh-add/)》

[《rsync同步的艺术》–linux命令五分钟系列之四十二](http://roclinux.cn/?p=2643)