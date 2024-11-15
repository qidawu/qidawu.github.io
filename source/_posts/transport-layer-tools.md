---
title: 传输层常用工具总结——netstat、ss、lsof
date: 2023-12-12 23:32:53
updated: 
tags: 计算机网络
typora-root-url: ..
---

# ss

![ss](/img/network/transport-layer/ss.png)

https://man7.org/linux/man-pages/man8/ss.8.html

```
ss [options] [ FILTER ]
其中，FILTER := [ state STATE-FILTER ] [ EXPRESSION ]
```

![ss](/img/network/transport-layer/ss_filter.png)

## 例子

```bash
# 列出当前 socket 统计信息
$ ss -s

# 显示所有已建立（established）的 HTTP、SMTP、SSH 连接
$ ss -o state established '( dport = :http or sport = :http )'
$ ss -o state established '( dport = :smtp or sport = :smtp )'
$ ss -o state established '( dport = :ssh or sport = :ssh )'

# 找出所有连接 X 服务器的进程
$ ss -x src /tmp/.X11-unix/*
```

```bash
# 统计指定进程 TCP 连接占用的端口
$ ss -tanp | grep 7059 | tr -s ' '  | cut -d ' ' -f 4 | sort -n | uniq -c

# 统计指定进程 TCP 连接状态
$ ss -tanp | grep 7059 | tr -s ' '  | cut -d ' ' -f 1 | sort -n | uniq -c

# 统计已建立 TCP 连接的 IP 数量
$ ss -tnp | grep 7059 | awk '{print $5}' | sort -n | uniq -c
```

### `ss`、`netstat`、`lsof` 命令的输出结果对比

`netstat` 是遍历 `/proc` 下面每个 PID 目录；

`ss` 直接读 `/proc/net` 下面的统计信息。所以 `ss` 执行的时候消耗资源以及消耗的时间都比 `netstat` 少很多。

![输出结果对比](/img/network/transport-layer/difference_between_ss_netstat_lsof.png)

# netstat

https://en.wikipedia.org/wiki/Netstat

https://linux.die.net/man/8/netstat

《[每天学一个 Linux 命令（65）：netstat](https://mp.weixin.qq.com/s/bZhZOoOhY-u9QALrWm-_Rw)》

## 例子

### Linux 端口占用统计

```bash
$ netstat -tanp | grep 7059 | tr -s ' '  | cut -d ' ' -f 4 | sort -n | uniq -c
```

### Mac 端口占用查询

```bash
$ netstat -anv | grep 7059
```

### Windows 端口占用查询

```bash
# 查询占用了 8080 端口的[进程号]（最后一列）
$ netstat -ano|findstr "8080"

# 找到[进程号]对应的[进程名称]
$ tasklist|findstr 7059

# 根据[进程名称]杀死进程
$ taskkill /f /t /im /javaw.exe
```

# lsof

![lsof](/img/network/transport-layer/lsof.png)

https://en.wikipedia.org/wiki/Lsof

https://linux.die.net/man/8/lsof

常用选项：

| 选项           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| **-n**         | 不显示主机名（host name），显示 IP（network number）。例如 localhost 显示为 127.0.0.1 |
| **-P**         | 不显示端口名（port name），显示端口号（port number）。例如 cslistener 显示为 9000 |
| **-a**         | AND 运算                                                     |
| **-p** *s*     | 筛选指定 PID                                                 |
| **-i** [...]   | 筛选指定条件，例如：TCP + 端口号                             |
| **-s** *[p:s]* | 筛选指定条件：例如：TCP (LISTEN)                             |

> **-i** *[`46`][`protocol`][@`hostname`|`hostaddr`][:`service`|`port`]*
>
> where:
>
> * `46` specifies the IP version, IPv4 or IPv6 that applies to the following address. '`6`' may be be specified only if the UNIX dialect supports IPv6.  If neither '`4`' nor  '`6`' is specified, the following address applies to all IP versions.
> * `protocol` is a protocol name - `TCP`, `UDP`
> * `hostname` is an Internet host name.  Unless a specific IP version is specified, open network files associated with host names of all versions will be selected.
> * `hostaddr` is a numeric Internet IPv4 address in dot form; or an IPv6 numeric address in colon form, enclosed in brackets, if the UNIX dialect supports IPv6.  When an IP version is selected, only its numeric addresses may be specified.
> * `service` is an /etc/services name - e.g., `smtp` or a list of them.
> * `port` is a port number, or a list of them.
> 
> Here are some sample addresses:
>
> ```
>     -i6 - IPv6 only
> 
>     -iTCP:25 - TCP and port 25
> 
>     -i@1.2.3.4 - Internet IPv4 host address 1.2.3.4
> 
>     -i@[3ffe:1ebc::1]:1234 - Internet IPv6 host address 3ffe:1ebc::1, port 1234
> 
>     -iUDP:who - UDP who service port
> 
>     -iTCP@lsof.itap:513 - TCP, port 513 and host name lsof.itap
> 
>     -iTCP@foo:1-10,smtp,99 - TCP, ports 1 through 10, service name smtp, port 99, host name foo
> 
>     -iTCP@bar:1-smtp - TCP, ports 1 through smtp, host bar
> 
>     -i:time - either TCP, UDP or UDPLITE time service port
> ```

> **-s** *[p:s]*
>
> When followed by a protocol *name* (p), either `TCP` or `UDP`, a colon (':') and a comma-separated protocol state name list, the option causes open TCP and UDP files to be 
> * excluded if their state **name**(s) are in the list (*s*) preceded by a '^'; 
>
> or
>
> * included if their state **name**(s) are not preceded by a '^'.
>
> For example, to list only network files with `TCP` state `LISTEN`, use:
>
> ````
> -iTCP -sTCP:LISTEN
> ````
>
> State names vary with UNIX dialects, so it's not possible to provide a complete list. Some common TCP state names are: `CLOSED`, `IDLE`, `BOUND`, `LISTEN`, `ESTABLISHED`, `SYN_SENT`, `SYN_RCDV`, `ESTABLISHED`, `CLOSE_WAIT`, `FIN_WAIT1`, `CLOSING`, `LAST_ACK`, `FIN_WAIT_2`, and `TIME_WAIT`.

## 例子

### 查询端口占用

```bash
# 筛选条件：端口号
$ lsof -nP -i:端口号

# 筛选条件：端口号 + TCP 协议
$ lsof -nP -iTCP:端口号

# 筛选条件：端口号 + TCP 协议 + LISTEN 状态
$ lsof -nP -iTCP:端口号 -sTCP:LISTEN
```

### 按 PID 查询

```bash
# 查看某个进程的 open files，等价于 /proc/PID/fd/
$ lsof -nP -p PID

# 按 TCP + 端口号 + PID 查询
$ lsof -nP -iTCP:端口号 -a -p PID

# 按 TCP (LISTEN) + PID 查询
$ lsof -nP -iTCP -sTCP:LISTEN -a -p PID

COMMAND  PID    USER   FD   TYPE     DEVICE SIZE/OFF NODE NAME
java    6276    dev   36u  IPv4 1097042515      0t0  TCP *:8720 (LISTEN)
java    6276    dev  150u  IPv4 1097044387      0t0  TCP *:8171 (LISTEN)
java    6276    dev  187u  IPv4 1097042576      0t0  TCP *:58077 (LISTEN)
java    6276    dev  202u  IPv4 1097042587      0t0  TCP *:8062 (LISTEN)
```

`lsof` 输出结果每一列的含义：

> COMMAND：进程的名称
> PID：进程标识符
> USER：进程所有者
> FD：文件描述符，应用程序通过文件描述符识别该文件。如 cwd、txt 等
> TYPE：文件类型，如 DIR、REG 等
> DEVICE：指定磁盘的名称
> SIZE：文件的大小
> NODE：索引节点（文件在磁盘上的标识〉
> NAME：打开文件的确切名称

![lsof 例子](/img/network/transport-layer/lsof_1.png)

![lsof 例子](/img/network/transport-layer/lsof_2.png)

![lsof 例子](/img/network/transport-layer/lsof_3.png)

# 参考

《[Linux 端口占用查询——netstat、ss、lsof | 良许 Linux](https://mp.weixin.qq.com/s/USzF4ngCRFiCU1IHHbiziA)》



