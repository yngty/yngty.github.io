---
title: I/O 多路复用
date: 2023-08-17 14:32:07
tags:
- socket
categories:
- Network
---

# select

`select` 实现多路复用的方式是，将已连接的 `socket` 都放到一个文件描述符集合，然后调用 `select`函数将文件描述符集合拷贝到内核里，让内核来检查是否有网络事件产生，通过遍历文件描述符集合的方式，当检查到有事件产生后，将此 `socket` 标记为可读或可写， 接着再把整个文件描述符集合拷贝回用户态里，然后用户态还需要再通过遍历的方法找到可读或可写的 `socket`，然后再对其处理。

所以，对于 `select` 这种方式，需要进行 **2 次「遍历」文件描述符集合**，一次是在内核态里，一个次是在用户态里 ，而且还会发生 **2 次「拷贝」文件描述符集合**，先从用户空间传入内核空间，由内核修改后，再传出到用户空间中。

## select 函数原型

```c
#include <sys/select.h>

int select(int nfds, fd_set *restrict readfds, fd_set *restrict writefds, fd_set *restrict errorfds, struct timeval *restrict timeout);
```
- 返回值 
    - 若有就绪描述符则为其数目，若超时则为 `0`，若出错则为 `-1`
- 参数
    - `maxfd`: 待测试的描述符基数，它的值是待测试的最大描述符加 `1`
    - `readfds`：读描述符集合
    - `writefds`：写描述符集合
    - `errorfds`：异常描述符集合
    - `timeout`: 超时设置

<!--more-->
## 操作描述集合

```c
void FD_ZERO(fd_set *fdset);　　　　　　
void FD_SET(int fd, fd_set *fdset);　　
void FD_CLR(int fd, fd_set *fdset);　　　
int  FD_ISSET(int fd, fd_set *fdset);
```
- `FD_ZERO` 清空描述符集合；
- `FD_SET` 向描述符集合增加 `fd`；
- `FD_CLR` 向描述符集合删除 `fd`；
- `FD_ISSET` 判断描述符集合中的 `fd` 是否有响应；

## 超时设置

`timeval` 结构体时间: 

```c
struct timeval {
  long   tv_sec; /* seconds */
  long   tv_usec; /* microseconds */
};
```
最后一个参数,可以设置 `3` 种值:

- 设置成空 `(NULL)`，表示如果没有 `I/O` 事件发生，则 `select` 一直等待下去
- 设置一个非零的值，等待超时时间阻塞返回
- `tv_sec` 和 `tv_usec` 都设置成 `0`，表示不等待，检测完毕立即返回

## 使用 🌰

在使用 `select` 时, 两个注意点:

- **描述符基数是当前最大描述符 +1；**
- **每次 `select` 调用完成之后，要重置待测试集合。**

```c
     int socket_fd = ...;
     fd_set ready_fds;

     while(true) {
        FD_ZERO(&read_fds);
        FD_SET(socket_fd, &read_fds);
        int rc = select(socket_fd + 1, &read_fds, NULL, NULL, NULL);
        if (rc == -1) {
            perror("select");
            return 1;
        }

         if (FD_ISSET(socket_fd, &read_fds)) {
            ...
         }

     }

```

**`select` 有一个缺点，那就是所支持的文件描述符的个数是有限的。在 `Linux` 系统中，`select` 的默认最大值为 `1024`**。

# `poll`

`poll` 可以突破 `select` 文件描述符的个数限制， 函数原型如下: 

```c
int poll(struct pollfd *fds, unsigned long nfds, int timeout);
```
- 返回值
    - 若有就绪描述符则为其数目，若超时则为 `0`，若出错则为 `-1`
- 参数
    - `fds`: `pollfd`数组
    - `nfds`: 描述 `fds`数组的大小
    - `timeout`: 超时设置, 单位 `ms`

## `pollfd`数组

`pollfd` 结构如下:

```c
struct pollfd {
    int    fd;       /* file descriptor */
    short  events;   /* events to look for */
    short  revents;  /* events returned */
};
```
- `fd`: 文件描述
- `events`: 待检测的事件类型
- `revents`:  响应的事件类型

`events` 类型的事件可以分为三大类。

第一类是可读事件，有以下几种：

```c
#define POLLIN          0x0001          /* any readable data available */
#define POLLPRI         0x0002          /* OOB/Urgent readable data */
#define POLLRDNORM      0x0040          /* non-OOB/URG data available */
#define POLLRDBAND      0x0080          /* OOB/Urgent readable data */
```
我们一般使用 `POLLIN`, 系统内核通知套接字缓冲区已准备好，通过 `read` 函数执行读操作不会被阻塞。

第二类是可写事件，有以下几种：

```c
#define POLLOUT         0x0004          /* file descriptor is writeable */
#define POLLWRNORM      POLLOUT         /* no write type differentiation */
#define POLLWRBAND      0x0100          /* OOB/Urgent data can be written */
```
我们一般使用 `POLLOUT`, 系统内核通知套接字缓冲区已准备好，通过 `write` 函数执行写操作不会被阻塞。


还有另一大类是错误事件，没有办法通过 `poll` 向系统内核递交检测请求，只能通过 `returned events`来加以检测:

```c
#define POLLERR    0x0008    /* 一些错误发送 */
#define POLLHUP    0x0010    /* 描述符挂起 */
#define POLLNVAL   0x0020    /* 请求的事件无效 */
```

**不想对某个 `pollfd` 结构进行事件检测**，可以把它对应的 `pollfd` 结构的 `fd` 成员设置成一个**负值**。这样，`poll` 函数将忽略这样的 `events` 事件，检测完成以后，所对应的`returned events`的成员值也将设置为 `0`。

## 超时设置 

- `< 0`，表示如果没有 `I/O` 事件发生，则 `poll` 一直等待下去
- `> 0`，等待超时时间阻塞返回
- `= 0`，表示不等待，检测完毕立即返回


## 使用 🌰

```c
#define INIT_SIZE 128
 
int main(int argc, char **argv) {
    int listen_fd, connected_fd;
    int ready_number;
    ssize_t n;
    char buf[MAXLINE];
    struct sockaddr_in client_addr;
 
    listen_fd = tcp_server_listen(SERV_PORT);
 
    // 初始化 pollfd 数组，这个数组的第一个元素是 listen_fd，其余的用来记录将要连接的 connect_fd
    struct pollfd event_set[INIT_SIZE];
    event_set[0].fd = listen_fd;
    event_set[0].events = POLLRDNORM;
 
    // 用 -1 表示这个数组位置还没有被占用
    int i;
    for (i = 1; i < INIT_SIZE; i++) {
        event_set[i].fd = -1;
    }
 
    for (;;) {
        if ((ready_number = poll(event_set, INIT_SIZE, -1)) < 0) {
            error(1, errno, "poll failed ");
        }
 
        if (event_set[0].revents & POLLRDNORM) {
            socklen_t client_len = sizeof(client_addr);
            connected_fd = accept(listen_fd, (struct sockaddr *) &client_addr, &client_len);
 
            // 找到一个可以记录该连接套接字的位置
            for (i = 1; i < INIT_SIZE; i++) {
                if (event_set[i].fd < 0) {
                    event_set[i].fd = connected_fd;
                    event_set[i].events = POLLRDNORM;
                    break;
                }
            }
 
            if (i == INIT_SIZE) {
                error(1, errno, "can not hold so many clients");
            }
 
            if (--ready_number <= 0)
                continue;
        }
 
        for (i = 1; i < INIT_SIZE; i++) {
            int socket_fd;
            if ((socket_fd = event_set[i].fd) < 0)
                continue;
            if (event_set[i].revents & (POLLRDNORM | POLLERR)) {
                if ((n = read(socket_fd, buf, MAXLINE)) > 0) {
                    if (write(socket_fd, buf, n) < 0) {
                        error(1, errno, "write error");
                    }
                } else if (n == 0 || errno == ECONNRESET) {
                    close(socket_fd);
                    event_set[i].fd = -1;
                } else {
                    error(1, errno, "read error");
                }
 
                if (--ready_number <= 0)
                    break;
            }
        }
    }
}
```
# epoll