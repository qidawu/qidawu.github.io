---
title: 网络编程系列（一）socket API 基础篇
date: 2023-12-25 21:48:49
updated:
tags: [网络编程, 计算机网络, C/C++]
mathjax: true
typora-root-url: ..
---

# socket 是什么

参考 [Berkeley套接字](https://zh.wikipedia.org/wiki/Berkeley套接字)

> Berkeley sockets 也称为 BSD Socket 。伯克利套接字的应用编程接口（API）是采用 C 语言的进程间通信的库，经常用在计算机网络间的通信。 BSD Socket 的应用编程接口已经是网络套接字的事实上的抽象标准。大多数其他程序语言使用一种相似的编程接口。
>
> BSD Socket 作为一种 API，允许不同主机或者同一个计算机上的不同进程之间的通信。它支持多种 I/O 设备和驱动，但是具体的实现是依赖操作系统的。这种接口对于 TCP/IP 是必不可少的，所以是互联网的基础技术之一。它最初是由加州伯克利大学为 Unix 系统开发出来的。所有现代的操作系统都实现了伯克利套接字接口，因为它已经是连接互联网的标准接口了。

![sockets](/img/network/socket/sockets.png)

# socket 流程

## UDP socket

![UDP socket](/img/network/socket/udp_socket.png)

## TCP socket

![TCP socket](/img/network/socket/tcp_socket.png)

### TCP 连接管理

TCP 连接管理（三次握手、四次挥手）的流程及状态机如下：

![TCP connection management](/img/network/transport-layer/tcp/connection-management/tcp_state_machine.jpg)

![TCP connection management](/img/network/transport-layer/tcp/connection-management/tcp_state_machine.png)

通过 Wireshark 抓包演示如下：在客户端 61151 向服务端 9000 发送两个报文的整个过程，发送之前须三次握手建立连接，发送完毕须四次挥手拆除连接：

![Wireshark](/img/network/transport-layer/tcp/connection-management/Wireshark.png)

# socket API

https://man7.org/linux/man-pages/man7/socket.7.html

| Syscall                    | Description                                                  | Blocking or not                       |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------- |
| `socket()`                 | creates an endpoint for communication and returns a file descriptor | NO                                    |
| `bind()`                   | bind a name to a socket                                      | NO                                    |
| `listen()`                 | listen for connections on a socket                           | NO                                    |
| `accept()`<br/>`accept4()` | creates a new file descriptor for an incoming connection     | YES or NO (`EAGAIN` or `EWOULDBLOCK`) |
| `connect()`                | initiate a connection on a socket                            | YES or NO (`EINPROGRESS`)             |
| `recv()`<br/>`send()`      | operations on a single file descriptor                       | YES or NO (`EAGAIN` or `EWOULDBLOCK`) |

## 服务端 API

### socket()

https://man7.org/linux/man-pages/man2/socket.2.html

> **SYNOPSIS**
>
> ```C
> #include <sys/socket.h>
> 
> // On success, a file descriptor for the new socket is returned. 
> // On error, -1 is returned, and errno is set appropriately.
> int socket(int domain, 
>            int type, 
>            int protocol);
> ```
>
> **DESCRIPTION**
>
> `socket()` creates an endpoint for communication and returns a file descriptor that refers to that endpoint.

`domain` 参数常用选项：

| Name       | Purpose                 | Man page                                          |
| ---------- | ----------------------- | ------------------------------------------------- |
| `AF_INET`  | IPv4 Internet protocols | ***[ip](https://man7.org/linux/man-pages/man7/ip.7.html)**(7)*     |
| `AF_INET6` | IPv6 Internet protocols | ***[ipv6](https://man7.org/linux/man-pages/man7/ipv6.7.html)**(7)* |

`type` 参数常用选项：

- `SOCK_DGRAM` 数据报套接字：Supports datagrams (connectionless, unreliable messages of a fixed maximum length).
- `SOCK_STREAM` 流式套接字：Provides sequenced, reliable, two-way, connection-based byte streams. An out-of-band data transmission mechanism may be supported.
- `SOCK_RAW` 原始套接字：Provides raw network protocol access.

![socket creation](/img/network/socket/socket_creation.png)

### bind()

https://man7.org/linux/man-pages/man2/bind.2.html

> **SYNOPSIS**
>
> ```C
> #include <sys/socket.h>
> 
> // On success, 0 is returned. 
> // On error, -1 is returned, and errno is set appropriately.
> int bind(int sockfd,
>         const struct sockaddr *addr,
>         socklen_t addrlen);
> ```
>
> **DESCRIPTION**
>
> The `bind()` assigns a local protocol address to a socket. With the Internet protocols, the address is the combination of an IPv4 or IPv6 address (32-bit or 128-bit) address along with a 16 bit TCP port number.

一般情况下，只有服务端 socket 需要指定端口号。而客户端 socket 为了避免发生端口号冲突，会由操作系统提供的随机[临时端口号](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)来运行，用户无须关心。但在某些情况下：

* 例如遇到防火墙只允许某些端口号出网，此时操作系统向客户端提供的端口号很可能会被客户端防火墙阻止。
* 某些协议（例如 NFS 协议）要求客户端程序仅在某个端口号上运行。

在这种情况下，我们需要显式或强制地使用 `bind()` 来为客户端 socket 指定特定端口号。

参考：[Explicitly assigning port number to client in Socket](https://geeksforgeeks.org/explicitly-assigning-port-number-client-socket/)

### listen()

https://man7.org/linux/man-pages/man2/listen.2.html

> **SYNOPSIS**
>
> ```C
> #include <sys/socket.h>
> 
> // On success, 0 is returned. 
> // On error, -1 is returned, and errno is set appropriately.
> int listen(int sockfd, 
>            int backlog);
> ```
>
> **DESCRIPTION**
>
> `listen()` marks the socket referred to by *sockfd* as a passive socket, that is, as a socket that will be used to accept incoming connection requests using `accept()`.
>
> The *backlog* argument defines the maximum length to which the queue of pending connections for *sockfd* may grow. If a connection request arrives when the queue is full, the client may receive an error with an indication of **ECONNREFUSED** or, if the underlying protocol supports retransmission, the request may be ignored so that a later reattempt at connection succeeds.
>
> The behavior of the *backlog* argument on TCP sockets changed with Linux 2.2. <mark>Now it specifies the queue length for *completely* established sockets waiting to be accepted, instead of the number of incomplete connection requests.</mark> The maximum length of the queue for incomplete sockets can be set using */proc/sys/net/ipv4/tcp_max_syn_backlog*. When syncookies are enabled there is no logical maximum length and this setting is ignored. See ***[tcp](https://man7.org/linux/man-pages/man7/tcp.7.html)**(7)* for more information.

![TCP socket](/img/network/socket/tcp_socket_2.png)

《[深入探索 Linux listen() 函数 backlog 的含义](https://blog.csdn.net/yangbodong22011/article/details/60399728)》

### accept()

https://man7.org/linux/man-pages/man2/accept.2.html

> **SYNOPSIS**
>
> ```c
> #include <sys/socket.h>
> 
> // On success, a nonnegative integer that is a file descriptor for the accepted socket is returned. 
> // On error, -1 is returned, and errno is set appropriately.
> int accept(int sockfd,
>           struct sockaddr *addr,
>           socklen_t *addrlen)
> ```
>
> **DESCRIPTION**
>
> The `accept()` system call is used with <mark>**connection-based** socket types (`SOCK_STREAM`, `SOCK_SEQPACKET`)</mark>. It extracts the first connection request on the queue of pending connections for the listening socket, *sockfd*, creates a new connected socket, and returns a new file descriptor referring to that socket. <mark>The newly created socket is not in the **listening** state.</mark> The original socket *sockfd* is unaffected by this call.
>
> If no pending connections are present on the queue, and the socket is not marked as nonblocking, `accept()` blocks the caller until a connection is present. If the socket is marked nonblocking and no pending connections are present on the queue, `accept()` fails with the error **EAGAIN** or **EWOULDBLOCK**.

## 客户端 API

### connect()

https://man7.org/linux/man-pages/man2/connect.2.html

> **SYNOPSIS**
>
> ```c
> #include <sys/socket.h>
> 
> // If the connection or binding succeeds, 0 is returned. 
> // On error, -1 is returned, and errno is set appropriately.
> int connect(int sockfd,
>             const struct sockaddr *addr,
>             socklen_t addrlen);
> ```
>
> **DESCRIPTION**
>
> The `connect()` function is used by a TCP client to establish a connection with a TCP server.
>
> The function returns `0` if the it succeeds in establishing a connection (i.e., successful TCP three-way handshake, `-1` otherwise.
>
> The client does not have to call `bind()` in Section before calling this function: the kernel will choose both an ephemeral port and the source IP if necessary.

### close()

https://man7.org/linux/man-pages/man2/close.2.html

> **SYNOPSIS**
>
> ```c
> #include <unistd.h>
> 
> int close(int fd);
> ```

用于完成四次挥手流程：

* 客户端首先调用 `close()` 主动关闭连接，这时 TCP 发送一个 FIN M；
* 服务端接收到 FIN M 之后，执行被动关闭，并发送 ACK M+1 对这个 FIN 进行确认；（它的接收也作为文件结束符 `EOF` 传递给应用进程（此时 `recv()` 返回值为 `0`），因为 FIN 的接收意味着应用进程在相应的连接上再也接收不到额外数据）
* 服务端之后调用 `close()` 主动关闭它的 socket，这导致它的 TCP 也发送一个 FIN N； 
* 客户端接收到 FIN N 之后，发送 ACK N+1 对这个 FIN 进行确认。 

这样每个方向上都有一个 FIN 和 ACK。

## 选项设置 API

> **SYNOPSIS**
>
> ```c
> int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
> ```
>
> ```c
> int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);
> ```
>
> **DESCRIPTION**
>
> **[getsockopt](https://man7.org/linux/man-pages/man2/getsockopt.2.html)**(2) and **[setsockopt](https://man7.org/linux/man-pages/man2/setsockopt.2.html)**(2) are used to set or get socket layer or protocol options.

`level` 常用选项如下：

socket layer

* [`SOL_SOCKET`](https://man7.org/linux/man-pages/man7/socket.7.html)：

protocol layer

* [`IPPROTO_TCP`](https://man7.org/linux/man-pages/man7/tcp.7.html)
* [`IPPROTO_IP`](https://man7.org/linux/man-pages/man7/ip.7.html)
* [`IPPROTO_IPV6`](https://man7.org/linux/man-pages/man7/ipv6.7.html)
* [`IPPROTO_ICMP`](https://man7.org/linux/man-pages/man7/icmp.7.html)
* [`SOL_PACKET`](https://man7.org/linux/man-pages/man7/packet.7.html)
* ...

> Using the `SOL_IP` socket options level isn't portable; BSD-based stacks use the `IPPROTO_IP` level.

### SOL_SOCKET

https://man7.org/linux/man-pages/man7/socket.7.html

#### 超时时间

**SO_RCVTIMEO** and **SO_SNDTIMEO**

> Specify the receiving or sending timeouts until reporting an error. The argument is a *struct timeval*. 
>
> Return value:
>
> * If an input or output function blocks for this period of time, and data has been sent or received, the return value of that function will be **the amount of data transferred**; 
> * if no data has been transferred and the timeout has been reached, then **-1** is returned with *errno* set to **EAGAIN** or **EWOULDBLOCK**, or **EINPROGRESS** (for **[connect](https://man7.org/linux/man-pages/man2/connect.2.html)**(2)) just as if the socket was specified to be nonblocking. 
>
> Timeout setting:
>
> * If the timeout is set to zero (the default) then the operation will never timeout. 
> * Timeouts only have effect for system calls that perform socket I/O (e.g., **[read](https://man7.org/linux/man-pages/man2/read.2.html)**(2), **[recvmsg](https://man7.org/linux/man-pages/man2/recvmsg.2.html)**(2), **[send](https://man7.org/linux/man-pages/man2/send.2.html)**(2), **[sendmsg](https://man7.org/linux/man-pages/man2/sendmsg.2.html)**(2)); 
> * timeouts have no effect for **[select](https://man7.org/linux/man-pages/man2/select.2.html)**(2), **[poll](https://man7.org/linux/man-pages/man2/poll.2.html)**(2), **[epoll_wait](https://man7.org/linux/man-pages/man2/epoll_wait.2.html)**(2), and so on.
>
> ```c
> setsockopt(clientfd, SOL_SOCKET, SO_RCVTIMEO, ...);
> setsockopt(clientfd, SOL_SOCKET, SO_SNDTIMEO, ...);
> ```

<[How to set socket timeout in C when making multiple connections?](https://stackoverflow.com/questions/4181784/how-to-set-socket-timeout-in-c-when-making-multiple-connections)>

<[Why there is no "write timeout" for the socket ?](https://stackoverflow.com/questions/48523304/java-socketwhy-is-there-is-no-write-timeout-for-the-socket)>

#### 缓冲区大小

**SO_RCVBUF**

> Sets or gets the maximum socket receive buffer in bytes. The kernel doubles this value (to allow space for bookkeeping overhead) when it is set using **[setsockopt](https://man7.org/linux/man-pages/man2/setsockopt.2.html)**(2), and this doubled value is returned by **[getsockopt](https://man7.org/linux/man-pages/man2/getsockopt.2.html)**(2). The default value is set by the */proc/sys/net/core/rmem_default* file, and the maximum allowed value is set by the */proc/sys/net/core/rmem_max* file. The minimum (doubled) value for this option is 256.

**SO_SNDBUF**

> Sets or gets the maximum socket send buffer in bytes. The kernel doubles this value (to allow space for bookkeeping overhead) when it is set using **[setsockopt](https://man7.org/linux/man-pages/man2/setsockopt.2.html)**(2), and this doubled value is returned by **[getsockopt](https://man7.org/linux/man-pages/man2/getsockopt.2.html)**(2). The default value is set by the */proc/sys/net/core/wmem_default* file and the maximum allowed value is set by the */proc/sys/net/core/wmem_max* file. The minimum (doubled) value for this option is 2048.

#### 地址重用

**SO_REUSEADDR**

> Indicates that the rules used in validating addresses supplied in a **[bind](https://man7.org/linux/man-pages/man2/bind.2.html)**(2) call should allow reuse of local addresses. For **AF_INET** sockets this means that a socket may bind, except when there is an active listening socket bound to the address. When the listening socket is bound to **INADDR_ANY** with a specific port then it is not possible to bind to this port for any local address. Argument is an integer boolean flag.
>
> ```C
> setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, ...);
> ```

#### 端口重用

**SO_REUSEPORT** (since Linux 3.9)

> ```c
> setsockopt(listenfd, SOL_SOCKET, SO_REUSEPORT, ...);
> ```

《[深入理解 Linux 端口重用这一特性——良许 Linux](https://mp.weixin.qq.com/s/wkxDLWmXInWq4fSPxNdDuQ)》

### IPPROTO_TCP

https://man7.org/linux/man-pages/man7/tcp.7.html

#### 长连接（TCP keepalive）

**SO_KEEPALIVE** 开启 TCP keepalive

> Enable sending of keep-alive messages on connection-oriented sockets. Expects an integer boolean flag.
>
> ```c
> setsockopt(clientfd, SOL_SOCKET, SO_KEEPALIVE, ...);
> ```



TCP keepalive 具体参数设置：

**TCP_KEEPCNT**: overrides `tcp_keepalive_probes`

**TCP_KEEPIDLE**: overrides `tcp_keepalive_time`

**TCP_KEEPINTVL**: overrides `tcp_keepalive_intvl`

> ```c
> setsockopt(clientfd, IPPROTO_TCP, TCP_KEEPCNT, ...);
> setsockopt(clientfd, IPPROTO_TCP, TCP_KEEPIDLE, ...);
> setsockopt(clientfd, IPPROTO_TCP, TCP_KEEPINTVL, ...);
> ```

《[传输层 TCP 协议保活机制总结（TCP 长连接）](/posts/transport-layer-tcp-keepalive/)》

#### 流量控制

**TCP_NODELAY**

> If set, disable the [Nagle algorithm](https://en.wikipedia.org/wiki/Nagle's_algorithm). This means that segments are always sent as soon as possible, even if there is only a small amount of data. When not set, data is buffered until there is a sufficient amount to send out, thereby avoiding the frequent sending of small packets, which results in poor utilization of the network.
>
> ```c
> setsockopt(clientfd, IPPROTO_TCP, TCP_NODELAY, ...);
> ```

## 读、写 API

成功建立连接之后，双方开始通过 `recv()` 和 `send()` 来读、写数据，就像往一个文件流里面写东西一样。

### recv()

https://man7.org/linux/man-pages/man2/recv.2.html

https://man7.org/linux/man-pages/man2/recvfrom.2.html

https://man7.org/linux/man-pages/man2/recvmsg.2.html

> receive a message from a socket
>
> * If no messages are available at the socket, the receive calls wait for a message to arrive, unless the socket is nonblocking (see **[fcntl](https://man7.org/linux/man-pages/man2/fcntl.2.html)**(2)), in which case the value -1 is returned and the external variable *errno* is set to **EAGAIN** or **EWOULDBLOCK**.
> * The **[select](https://man7.org/linux/man-pages/man2/select.2.html)**(2) or **[poll](https://man7.org/linux/man-pages/man2/poll.2.html)**(2) call may be used to determine when more data arrives.

```c
#include <sys/socket.h>

// return value: 
// n bytes received
// 0 when the peer has performed an orderly shutdown
// -1 if an error occurred (nonblocking)
```

### send()

https://man7.org/linux/man-pages/man2/send.2.html

https://man7.org/linux/man-pages/man2/sendto.2.html

https://man7.org/linux/man-pages/man2/sendmsg.2.html

> send a message on a socket
>
> * When the message does not fit into the send buffer of the socket, **send**() normally blocks, unless the socket has been placed in nonblocking I/O mode. In nonblocking mode it would fail with the error **EAGAIN** or **EWOULDBLOCK** in this case.
> * The **[select](https://man7.org/linux/man-pages/man2/select.2.html)**(2) call may be used to determine when it is possible to send more data.

```c
#include <sys/socket.h>

// return value: 
// n characters sent
// -1 if an error occurred
```

# nonblocking I/O on sockets

默认设置下，socket 为同步阻塞 I/O 模型，下列系统调用会引发阻塞：

| Call type               | Socket state                | blocking                | Nonblocking                                      |
| ----------------------- | --------------------------- | ----------------------- | ------------------------------------------------ |
| Types of `recv()` calls | Input is available          | Immediate return        | Immediate return                                 |
|                         | No input is available       | Wait for input          | Immediate return with `EWOULDBLOCK` error number |
| Types of `send()` calls | Output buffers available    | Immediate return        | Immediate return                                 |
|                         | No output buffers available | Wait for output buffers | Immediate return with `EWOULDBLOCK` error number |
| `accept()` call         | New connection              | Immediate return        | Immediate return                                 |
|                         | No connections queued       | Wait for new connection | Immediate return with `EWOULDBLOCK` error number |
| `connect()` call        |                             | Wait                    | Immediate return with `EINPROGRESS` error number |

有几种设置方式可以将 socket 设置为同步非阻塞 I/O 模型：

## 设置方式一

使用 **[fcntl](https://man7.org/linux/man-pages/man2/fcntl.2.html)**(2) + `O_NONBLOCK` 参数。POSIX 标准，版本兼容性最好，推荐使用：

```C
int flags = fcntl(listenfd, F_GETFL, 0);
// setting the O_NONBLOCK flag on a socket file descriptor using fcntl()
int status = fcntl(listenfd, F_SETFL, flags | O_NONBLOCK);
```

使用 **[ioctl](https://man7.org/linux/man-pages/man2/ioctl.2.html)**(2) + `FIONBIO` 参数。版本兼容性较弱，不建议使用：

```C
int on = 1;
/*************************************************************/
/* Set socket to be nonblocking. All of the sockets for      */
/* the incoming connections will also be nonblocking since   */
/* they will inherit that state from the listening socket.   */
/*************************************************************/
int status = ioctl(listenfd, FIONBIO, (char *)&on);
```

## 设置方式二

从 Linux 2.6.27 开始，也可以使用 **[socket](https://man7.org/linux/man-pages/man2/socket.2.html)**(2) 和 **[accept4](https://man7.org/linux/man-pages/man2/accept.2.html)**(2) 直接创建非阻塞 socket：

```C
int listenfd, clientfd;

// 从 Linux 2.6.27 开始， socket() 的 type 参数还有第二个用途：除了指定 socket 类型之外，它还可以与值 SOCK_NONBLOCK 按位或，以修改 socket() 的行为
listenfd = socket(AF_INET, SOCK_STREAM | SOCK_NONBLOCK, 0);

// bind()、listen()

// nonblocking
// accept4() 相比 accept() 新增了第四个参数：flags
clientfd = accept4(listenfd, ... , SOCK_NONBLOCK);
```

上述几个文件描述符 fd 的基础概念：

* listenfd：服务端创建的 socket，`bind()` 地址和端口，调用 `listen()` 函数发起侦听的一端；
* clientfd：服务端调用 `accept()` 函数接受新连接，由 `accept()` 函数返回的 socket。`accept()` 函数并不参与三次握手过程，`accept()` 函数从已经完成连接的 backlog 队列中取出连接，并返回 clientfd；
* connfd：客户端创建的 socket，调用 `connect()` 函数主动发起连接（三次握手）的一端。

最后客户端与服务端分别通过 connfd 和 clientfd 进行通信（调用 `send()` 或者 `recv()` 函数）。

![fd](/img/network/socket/fd.jpg)

# socket 代码样例

https://cs.dartmouth.edu/~campbell/cs60/socketprogramming.html

https://www.geeksforgeeks.org/socket-programming-cc/

https://github.com/qidawu/cpp-data-structure/tree/main/socket

# 参考

https://en.wikipedia.org/wiki/Network_socket

https://en.wikipedia.org/wiki/Berkeley_sockets

[The Berkeley Sockets API](https://csperkins.org/teaching/2007-2008/networked-systems/lecture04.pdf)

<[Client/server socket programs: Blocking, nonblocking, and asynchronous socket calls | IBM](https://www.ibm.com/docs/en/zos/3.1.0?topic=otap-clientserver-socket-programs-blocking-nonblocking-asynchronous-socket-calls)>

