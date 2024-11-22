---
title: 传输层 TCP 协议保活机制总结（TCP 长连接）
date: 2023-05-25 17:48:49
updated:
tags: 计算机网络
typora-root-url: ..
---

# 什么是 TCP keepalive?

> The keepalive concept is very simple: when you set up a TCP connection, you associate a set of timers.

# 为什么使用 TCP keepalive ?

TCP 是一个基于连接的协议，其连接状态是由一个**状态机**进行维护，连接完毕后，双方都会处于 `established` 状态，默认情况下，这之后的状态并不会主动进行变化。这意味着如果上层不进行任何调用，一直使 TCP 连接空闲，那么这个连接虽然没有任何数据，但仍然保持连接状态，一天、一星期、甚至一个月，即使在这期间：

- 对端主机系统奔溃
- 中间路由原因（如奔溃重启、NAT 转换表移除该记录）

# TCP keepalive 的两个目标

## Checking for dead peers

```
    _____                                                     _____ 
   |     |                                                   |     | 
   |  A  |                                                   |  B  | 
   |_____|                                                   |_____| 
      ^                                                         ^ 
      |--->--->--->-------------- SYN -------------->--->--->---| 
      |---<---<---<------------ SYN/ACK ------------<---<---<---| 
      |--->--->--->-------------- ACK -------------->--->--->---| 
      |                                                         | 
      |                                       system crash ---> X 
      | 
      |                                     system restart ---> ^ 
      |                                                         | 
      |--->--->--->-------------- PSH -------------->--->--->---| 
      |---<---<---<-------------- RST --------------<---<---<---| 
      |                                                         |
```

## Preventing disconnection due to network inactivity

```
    _____           _____                                     _____ 
   |     |         |     |                                   |     | 
   |  A  |         | NAT |                                   |  B  | 
   |_____|         |_____|                                   |_____| 
      ^               ^                                         ^ 
      |--->--->--->---|----------- SYN ------------->--->--->---| 
      |---<---<---<---|--------- SYN/ACK -----------<---<---<---| 
      |--->--->--->---|----------- ACK ------------->--->--->---| 
      |               |                                         | 
      |               | <--- connection deleted from NAT table  | 
      |               |                                         | 
      |--->- PSH ->---| <--- invalid connection                 | 
      |               |                                         |
```

# Linux 使用 TCP keepalive

Linux 有内置的 keepalive 支持，配置如下：

```bash
$ cat /proc/sys/net/ipv4/tcp_keepalive_time 
7200 
$ cat /proc/sys/net/ipv4/tcp_keepalive_probes 
9 
$ cat /proc/sys/net/ipv4/tcp_keepalive_intvl 
75 

$ sysctl -a | grep keepalive 
net.ipv4.tcp_keepalive_time = 7200 
net.ipv4.tcp_keepalive_probes = 9 
net.ipv4.tcp_keepalive_intvl = 75

# 查看 TCP 协议的一些描述和参数定义（man 命令使用手册共 9 章，TCP 的帮助手册位于第 7 章）
$ man tcp
$ man 7 tcp
```

上述参数表示：默认情况下一条 TCP 连接在 2 小时（7200 秒）都没有报文交换后，会开始进行保活探测，若再经过 9*75 秒 = 11 分钟 15 秒的循环探测都未收到探测响应（即共计：2 小时 11 分钟 15 秒），就会自动断开 TCP 连接。

在探测过程中，对方主机会处于以下四种状态之一：

| 状态                                   | 处理方式                                                     |                  |
| -------------------------------------- | ------------------------------------------------------------ | ---------------- |
| 对方主机仍在工作，并且可达             | TCP 连接正常，将 keepalive 计时器重置                        |                  |
| 对方主机崩溃，并且已经重启             | 重启后原连接已失效，对方主机由于不认识探测报文，会响应重置报文段（RST） | TCP 连接断开重连 |
| 对方主机已崩溃（已关机或者正在重启）   | TCP 连接不正常，请求端经过指定次数的探测依然没得到响应       | TCP 连接断开重连 |
| 对方主机仍在工作，但由于网络原因不可达 | TCP 连接不正常，请求端经过指定次数的探测依然没得到响应       | TCP 连接断开重连 |

# 应用程序使用 TCP keepalive

![TCP keepalive configuration](/img/network/transport-layer/tcp/tcp_keepalive_configuration.png)

# 参考

《TCP/IP 详解 卷 1: 协议》第二版，第 17 章：TCP Keepalive 保活机制》

https://tldp.org/HOWTO/TCP-Keepalive-HOWTO/index.html

https://en.wikipedia.org/wiki/Keepalive

https://linux.die.net/man/7/tcp



《[HTTP 长连接和 TCP 长连接有区别？图解](https://mp.weixin.qq.com/s/R-gnBNrB8FQe7FJHtmbKXw)》

《[HTTP keep-alive 和 TCP keepalive 的区别，你了解吗？](https://zhuanlan.zhihu.com/p/224595048)》
