---
title: 网络编程系列（二）socket I/O 模型对比
date: 2023-12-27 21:48:49
updated:
tags: [网络编程, 计算机网络, C/C++]
mathjax: true
typora-root-url: ..
---

> 原则13：从硬件、操作系统到你使用的编程语言等多方面深入了解服务器的工作原理。优化 IO 操作的效率是一个良好架构的首要任务。
>
> 原则15：如果你的设计是基于事件驱动的非阻塞架构，那就不要阻塞线程或者在线程中执行 IO 操作。一旦这样做，系统将慢如蜗牛。

本文主要介绍和对比 socket 常用的几种 I/O 模型：

![Comparison of the five I/O model](/img/network/socket/comparison_of_the_five_io_models.png)

# 同步阻塞 I/O

默认设置下，socket 为同步阻塞 I/O 模型，需**阻塞等待**新的连接、新的数据报到达：

![Blocking I/O model](/img/network/socket/blocking_io_model.png)

有两种实现方式：

## 单线程模型

![Single Thread](/img/network/socket/single_thread.jpg)

优点：

- 编程实现简单。

缺点：

- 采用**先来先服务（FCFS）原则进行排队处理**，处理效率低下：因为一次只处理 backlog 中的一个连接，直到对方 `close()` 发送 EOF 关闭连接后，才能继续处理 backlog 中的下一个连接。如果该连接一直未关闭，则会阻塞主线程使其一直等待该连接新的数据报到达，导致：
  - 客户端的请求响应时间很长，体验感差，因为主线程无法及时处理新到达的连接；
  - 服务端的 CPU 利用率（CPU utilization）低下，因为很大可能，时间都消耗在阻塞等待上了：

![Single Thread blocking](/img/network/socket/single_thread_blocking.png)

### 样例代码

https://github.com/qidawu/cpp-data-structure/blob/main/socket/tcpIterativeServer.c

### 流程图

同步阻塞 I/O + 单线程模型的服务端流程如下：

![Blocking I/O workflow](/img/network/socket/blocking_io_workflow.jpg)

## 多线程模型

多线程模型（Thread-based architecture / Thread per Request Model）

![Multi Thread](/img/network/socket/multi_thread.jpg)

优点：

- 线程之间互不干扰、互不影响

缺点：

- 只适合于并发量不大的场景，建议使用**线程池**实现，否则：
  - 线程的创建、销毁开销大
  - 线程本身需要占用内存资源（-Xss 栈）
  - 线程数有上限（`ulimit -u`）
  - 线程间的上下文频繁切换开销大
- 各个工作线程内仍然存在阻塞等待的问题，CPU 利用率（CPU utilization）低下：

![Multi Thread blocking](/img/network/socket/multi_thread_blocking.png)

# 同步非阻塞 I/O

有几种设置方式可以将 socket 设置为同步非阻塞 I/O 模型。设置后，需忙轮询（polling）等待新的连接、新的数据报到达：

> Then all operations that would block will (usually) return with **EAGAIN** or **EWOULDBLOCK** (operation should be retried later); **[connect](https://man7.org/linux/man-pages/man2/connect.2.html)**(2) will return **EINPROGRESS** error. The user can then wait for various events via **[poll](https://man7.org/linux/man-pages/man2/poll.2.html)**(2) or **[select](https://man7.org/linux/man-pages/man2/select.2.html)**(2).

![Nonblocking I/O model](/img/network/socket/nonblocking_io_model.png)

优点：

- 解决了 BIO 的阻塞问题，无需开启过多线程

缺点：

- 忙等浪费 CPU 资源：主线程需要不断轮询，以判断是否有新的连接到达（以便 `accept()`），或者各个已建立的连接是否有新的数据报到达（以便 `recv()`）。反复多次无谓的系统调用，浪费系统资源。

## 样例代码

## 流程图

同步非阻塞 I/O + 单线程模型的服务端流程如下：

![Nonblocking I/O workflow](/img/network/socket/nonblocking_io_workflow.jpg)

#  I/O 多路复用

什么是多路复用（[Multiplexing](https://en.wikipedia.org/wiki/Multiplexing)）？

> In [telecommunications](https://en.wikipedia.org/wiki/Telecommunications) and [computer networking](https://en.wikipedia.org/wiki/Computer_network), **multiplexing** (sometimes contracted to **muxing**) is a method by which multiple analog or digital signals are combined into one signal over a [shared medium](https://en.wikipedia.org/wiki/Shared_medium). The aim is to share a scarce resource – a physical [transmission medium](https://en.wikipedia.org/wiki/Transmission_medium).
>
> A device that performs the multiplexing is called a **[multiplexer](https://en.wikipedia.org/wiki/Multiplexer) (MUX)**, and a device that performs the reverse process is called a **[demultiplexer](https://en.wikipedia.org/wiki/Demultiplexer) (DEMUX or DMX)**.
>
> In [computing](https://en.wikipedia.org/wiki/Computing), **I/O multiplexing** can also be used to refer to the concept of processing multiple [input/output](https://en.wikipedia.org/wiki/Input/output) [events](https://en.wikipedia.org/wiki/Event_(computing)) from a single [event loop](https://en.wikipedia.org/wiki/Event_loop), with system calls like [poll](https://en.wikipedia.org/wiki/Poll_(Unix))[[1\]](https://en.wikipedia.org/wiki/Multiplexing#cite_note-1) and [select (Unix)](https://en.wikipedia.org/wiki/Select_(Unix)).[[2\]](https://en.wikipedia.org/wiki/Multiplexing#cite_note-2)

为了减少忙轮询（polling）带来的 CPU 资源消耗，可以使用 I/O 多路复用，同时监视多个文件描述符：

![I/O multiplexing API](/img/network/socket/io_multiplexing_api.webp)

一旦一个或多个文件描述符就绪，就通知程序进行相应的操作：

![I/O multiplexing model](/img/network/socket/io_multiplexing_model.png)

适用场景：

- 基于 event loop 模型的 [Reactor pattern](https://en.wikipedia.org/wiki/Reactor_pattern)

优点：

- 减少盲目轮询带来的 CPU 资源消耗：无需 n 次系统调用（`accept()`、`recv()`），使用 I/O 多路复用只需 1 次系统调用，即可同时监视多个文件描述符是否就绪。

I/O 多路复用在各平台的 API 如下：

> The classical functions for I/O multiplexing are [`select`](https://man7.org/linux/man-pages/man2/select.2.html) and [`poll`](https://man7.org/linux/man-pages/man2/poll.2.html). Due to several limitations, modern operating systems provide advanced (non-portable) variants to these:
>
> - Windows provides [I/O completion ports (IOCP)](https://learn.microsoft.com/en-us/windows/win32/fileio/i-o-completion-ports).
> - BSD provides [`kqueue`](https://www.freebsd.org/cgi/man.cgi?kqueue).
> - Linux provides [`epoll`](https://man7.org/linux/man-pages/man7/epoll.7.html).
> - Solaris provides `evport`.

I/O 多路复用 API 对比表格：

| 演进       | 诞生时间             | 是否 POSIX 标准  | API 易用程度 | 能够监视的事件 | 能够监视的文件描述符数量 | 性能（用户如何获取已就绪的文件描述符集合） |
| ---------- | -------------------- | ------------------------ | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `select()` | 上世纪 80 年代       | 是      | 低（每次 `select()` 完都需重置 `fd_set`） | `readfds`<br/>`writefds`<br/>`exceptfds` | < 1024 | 低（轮询实现） |
| `poll()`   | 1997 年              | 是             | 中 | `POLLIN`<br/>`POLLOUT`<br/>`POLLPRI`<br/>`POLLERR`<br/>`POLLHUP`<br/>`POLLRDHUP` | ≥ 1024 | 低（轮询实现） |
| `epoll`    | 2002 年 Linux 2.5.44 | 否 | 高 | `EPOLLIN`<br/>`EPOLLOUT`<br/>`EPOLLPRI`<br/>`EPOLLERR`<br/>`EPOLLHUP`<br/>`EPOLLRDHUP` | ≥ 1024 | 高（红黑树+双向链表） |

## select()

https://man7.org/linux/man-pages/man2/select.2.html

> synchronous I/O multiplexing
>
> **SYNOPSIS**
>
> ```c
> #include <sys/select.h>
> 
> // return value: n, 0, -1
> // On success, number of file descriptors contained in the three returned descriptor sets
> // 0 if the timeout expired before any file descriptors became ready.
> // On error, -1 is returned, and errno is set to indicate the error
> int select(int nfds, 
>      fd_set *readfds, 
>      fd_set *writefds, 
>      fd_set *exceptfds, 
>      struct timeval *timeout);
> ```
>
> **DESCRIPTION**
>
> **⚠️WARNING**: `select()` can monitor only file descriptors numbers that are <mark>less than `FD_SETSIZE` (1024)</mark>—an unreasonably low limit for many modern applications—and this limitation will not change. All modern applications should instead use [`poll(2)`](https://man7.org/linux/man-pages/man2/poll.2.html) or [`epoll(7)`](https://man7.org/linux/man-pages/man7/epoll.7.html), which do not suffer this limitation.
>
> <mark>`select()` allows a program to monitor **multiple file descriptors**, waiting until one or more of the file descriptors become "ready" for some class of I/O operation (e.g., input possible).</mark>  A file descriptor is considered ready if it is possible to perform a corresponding I/O operation (e.g., [`read(2)`](https://man7.org/linux/man-pages/man2/read.2.html), or a sufficiently small [`write(2)`](https://man7.org/linux/man-pages/man2/write.2.html)) without **blocking**.

> Four macros are provided to manipulate the sets
>
> ```c
> // clears a set
> void FD_ZERO(fd_set *set);
> // add a given file descriptor from a set
> void FD_SET(int fd, fd_set *set);
> // remove a given file descriptor from a set
> void FD_CLR(int fd, fd_set *set);
> // tests to see if a file descriptor is part of the set
> int  FD_ISSET(int fd, fd_set *set);
> ```
>
> this is useful after `select()` returns.

### 样例代码

样例代码参考：

https://man7.org/linux/man-pages/man2/select.2.html#EXAMPLES

<[Example: Nonblocking I/O and select() | IBM](https://www.ibm.com/docs/en/i/7.5?topic=designs-example-nonblocking-io-select)>

### 流程图

服务端流程如下：

![select workflow](/img/network/socket/select_workflow.gif)

## poll()

https://man7.org/linux/man-pages/man2/poll.2.html#EXAMPLES

> wait for some event on a file descriptor
>
> **SYNOPSIS**
>
> ```c
> #include <poll.h>
>
> int poll(struct pollfd *fds, nfds_t nfds, int timeout);
> ```
>
> **DESCRIPTION**
>
> `poll()` performs a similar task to [`select(2)`](https://man7.org/linux/man-pages/man2/select.2.html): it waits for one of a set of file descriptors to become ready to perform I/O.  The Linux-specific [`epoll(7)`](https://man7.org/linux/man-pages/man7/epoll.7.html) API performs a similar task, but offers features beyond those found in `poll()`.

### 样例代码

样例代码参考：

https://man7.org/linux/man-pages/man2/poll.2.html#EXAMPLES

<[Using poll() instead of select() | IBM](https://www.ibm.com/docs/en/i/7.5?topic=designs-using-poll-instead-select)>

### 流程图

## epoll

https://man7.org/linux/man-pages/man7/epoll.7.html

> `/proc/sys/fs/epoll/max_user_watches` (since Linux 2.6.28)
>
> This specifies a limit on the total number of file descriptors that a user can register across all epoll instances on the system.

### API

#### epoll_create

https://man7.org/linux/man-pages/man2/epoll_create.2.html

> open an epoll file descriptor
>
> ```c
> #include <sys/epoll.h>
> 
> // On success, return a file descriptor (a nonnegative integer).
> // On error, returns -1 and errno is set to indicate the error.
> int epoll_create(int size);
> int epoll_create1(int flags);
> ```

用法如下：creates a new `epoll` instance and returns a file descriptor referring to that instance.

```c
int epfd = epoll_create1(0);
```

#### epoll_ctl

https://man7.org/linux/man-pages/man2/epoll_ctl.2.html

> control interface for an epoll file descriptor
>
> ```c
> #include <sys/epoll.h>
> 
> // On success, returns 0.
> // On error, returns -1 and errno is set to indicate the error.
> int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
> ```
>
> `op` 参数支持三种操作：
>
> | op              | Description            |
> | --------------- | ---------------------- |
> | `EPOLL_CTL_ADD` | 增加被监视的文件描述符 |
> | `EPOLL_CTL_DEL` | 删除被监视的文件描述符 |
> | `EPOLL_CTL_MOD` | 修改被监视的文件描述符 |
>
> `event` 参数（结构参考：[`struct epoll_event`](https://man7.org/linux/man-pages/man3/epoll_event.3type.html)）用于描述文件描述符 `fd`。`event` 参数的 `events` 成员变量包含两部分含义：event types 和 input flags
>
> 常用的 event types：
>
> | Event types | Description                                                  |
> | ----------- | ------------------------------------------------------------ |
> | `EPOLLIN`   | The associated file is available for [`read(2)`](https://man7.org/linux/man-pages/man2/read.2.html) operations. |
> | `EPOLLOUT`  | The associated file is available for [`write(2)`](https://man7.org/linux/man-pages/man2/write.2.html) operations. |
> | `EPOLLERR`  | Error condition happened on the associated file descriptor. This event is also reported for the write end of a pipe when the read end has been closed.<br/>[epoll_wait(2)](https://man7.org/linux/man-pages/man2/epoll_wait.2.html) will always **report** for this event; it is not necessary to set it in `events` when calling `epoll_ctl()`. |
> | `EPOLLHUP`  | Hang up happened on the associated file descriptor.<br/>[epoll_wait(2)](https://man7.org/linux/man-pages/man2/epoll_wait.2.html) will always **wait** for this event; it is not necessary to set it in `events` when calling `epoll_ctl()`. |
>
> 常用的 input flags：
>
> | Input flags | Description                                                  |
> | ----------- | ------------------------------------------------------------ |
> | `EPOLLET`   | Requests **edge-triggered** notification for the associated file descriptor.  The default behavior for epoll is **level-triggered**. See [epoll(7)](https://man7.org/linux/man-pages/man7/epoll.7.html) for more detailed information about edge-triggered and level-triggered notification. |
>

用法如下：adds items to the *interest* list of the `epoll` instance

```c
struct epoll_event event;
event.data.fd = listenfd;
event.events = EPOLLIN | EPOLLET;  // edge-triggered mode
s = epoll_ctl(epfd, EPOLL_CTL_ADD, listenfd, &event);
```

#### epoll_wait

https://man7.org/linux/man-pages/man2/epoll_wait.2.html

> wait for an I/O event on an epoll file descriptor
>
> ```c
> #include <sys/epoll.h>
> 
> // return value: n, 0, -1
> // On success, returns the number of file descriptors ready for the requested I/O operation, 
> //             or 0 if no file descriptor became ready during the requested timeout milliseconds.
> // On error, returns -1 and errno is set to indicate the error.
> int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
> ```
>
> The buffer pointed to by `events` is used to return information from the *ready* list about file descriptors in the *interest* list that have some events available.
>
> A call to `epoll_wait()` will block until either:
>
> * a file descriptor delivers an event;
> * the call is interrupted by a signal handler; or
> * the timeout expires.
>

用法如下：fetchs items from the *ready* list of the `epoll` instance

```c
struct epoll_event events[MAXEVENTS];

/* The event loop */
while (1)
{
  int n, i;

  n = epoll_wait(epfd, events, MAXEVENTS, -1);
  for (i = 0; i < n; i++)
  {
    if (listenfd == events[i].data.fd) {
      /* We have a notification on the listening socket, which
         means one or more incoming connections. */
      while (1) {
        // accept() a new connection
        // epoll_ctl() add to the *interest* list
      }
    }
    else if (events[i].events & EPOLLIN) {
      /* We have data on the fd waiting to be read. Read and
         display it. We must read whatever data is available
         completely, as we are running in edge-triggered mode
         and won't get a notification again for the same
         data. */
      while (1) {
        // recv()
      }
    }
  }
}
```

### 样例代码

样例代码参考：《[如何使用 epoll? 一个 C 语言实例](https://www.oschina.net/translate/how-to-use-epoll-a-complete-example-in-c?cmp)》

### 底层实现

`epoll` 实例由两部分组成：

* The *interest* list：**被监视的**文件描述符集合，由 `epoll_ctl()` 增删改，数据结构为红黑树。
* The *ready* list：**已就绪的**文件描述符集合，由 `epoll_wait()` 返回，数据结构为双向链表。

> The central concept of the `epoll` API is the `epoll` *instance*, an in-kernel data structure which, from a user-space perspective, can be considered as a container for two lists:
>
> * The *interest* list (sometimes also called the `epoll` set): the set of file descriptors that the process has registered an interest in monitoring.
> * The *ready* list: the set of file descriptors that are "ready" for I/O.  The ready list is a subset of (or, more precisely, a set of references to) the file descriptors in the *interest* list. The *ready* list is dynamically populated by the kernel as a result of I/O activity on those file descriptors.

![eventpoll 结构](/img/network/socket/epoll_data_structure.webp)

`epoll` 这种设计能显著提高程序在大量并发连接中只有少量活跃的情况下的系统 CPU 利用率（CPU utilization），因为在 `epoll_wait()` 获取事件的时候，它无须遍历整个被监视的文件描述符集合（The *interest* list），只要遍历那些被内核 I/O 事件异步唤醒而加入 Ready 队列的文件描述符集合（The *ready* list）即可。

![eventpoll 结构](/img/network/socket/epoll_data_structure_2.webp)

### 流程图

服务端流程如下：

![epoll workflow](/img/network/socket/epoll_workflow.png)

抽象来看，使用 I/O 多路复用可以实现*基于单线程的事件循环模型*（Single Threaded [Event Loop](https://en.wikipedia.org/wiki/Event_loop) Model）：

![event loop](/img/network/socket/event_loop.png)

站在客户端、服务端的角度，整体交互流程如下：

![epoll workflow](/img/network/socket/epoll_workflow_2.png)

## 使用场景

Netty

* Spring WebFlux、Dubbo、Vert.x
* Apache Kafka、RocketMQ、ElasticSearch、Zookeeper

Redis、Nginx、Node.js、…

通过命令 `strace` 可以查看应用程序使用了哪些系统调用。例如查看 Redis server 如何使用 epoll API：

![epoll of redis](/img/network/socket/epoll_of_redis.jpg)

《[Redis 是如何建立连接和处理命令的 | 阿里技术](https://mp.weixin.qq.com/s/FuroHE04BMpwbY0w9Pyfgw)》

# 参考

I/O multiplexing

* https://en.wikipedia.org/wiki/Select_(Unix)
* https://en.wikipedia.org/wiki/Poll_(Unix)
* https://en.wikipedia.org/wiki/Epoll
* https://en.wikipedia.org/wiki/Kqueue
* https://en.wikipedia.org/wiki/Input/output_completion_port

Event loop

* https://en.wikipedia.org/wiki/Event_loop
* 《[什么是 Event Loop？| 阮一峰](https://www.ruanyifeng.com/blog/2013/10/event_loop.html)》
* Thread per Request Model vs. Single Threaded Event Loop Model![event loop](/img/network/socket/event_loop_in_nodejs.jpg)

Reactor

* https://en.wikipedia.org/wiki/Reactor_pattern

http://www.kegel.com/c10k.html

I/O 多路复用系列

* 《[还在用多线程？试试 Linux select 这个‘神操作’吧！](https://mp.weixin.qq.com/s/sRXjRUZS1BVx1ZtBsZifug)》
* 《[Linux IO 复用大揭秘：select 落伍了？poll 才是高效之选！](https://mp.weixin.qq.com/s/aOplK2JkUJPm--Dwabsq7g)》
* 《[别再纠结 select 和 poll 了！epoll 才是 I/O 复用的顶流担当！](https://mp.weixin.qq.com/s/fE6xROVb5c4PrePOFrAbeA)》

《[Linux 高性能服务 epoll 的本质，真的不简单 | 良许 Linux](https://mp.weixin.qq.com/s/a3qjnucpBHfIoRKlSOrQ2w)》

《[从 Linux 源码角度看 epoll，透过现象看本质 | 良许 Linux](https://mp.weixin.qq.com/s/IK3WJP-KLgZ5X1YtjfDG7w)》

《Netty 权威指南》李林峰
