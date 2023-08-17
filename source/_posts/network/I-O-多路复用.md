---
title: I/O 多路复用
date: 2023-08-17 14:32:07
tags:
- socket
categories:
- Network
---

# select

`select` 实现多路复用的方式是，将已连接的 `socket` 都放到一个文件描述符集合，然后调用 `select`函数将文件描述符集合拷贝到内核里，让内核来检查是否有网络事件产生，通过遍历文件描述符集合的方式，当检查到有事件产生后，将此 `socket` 标记为可读或可写， 接着再把整个文件描述符集合拷贝回用户态里，然后用户态还需要再通过遍历的方法找到可读或可写的 `Socket`，然后再对其处理。

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

## 使用例子

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

````

**`select` 有一个缺点，那就是所支持的文件描述符的个数是有限的。在 `Linux` 系统中，`select` 的默认最大值为 `1024`**。

# poll

`poll` 
```c
struct pollfd {
    int    fd;       /* file descriptor */
    short  events;   /* events to look for */
    short  revents;  /* events returned */
};

int poll(struct pollfd *fds, unsigned long nfds, int timeout);

#define    POLLIN    0x0001    /* any readable data available */
#define    POLLPRI   0x0002    /* OOB/Urgent readable data */
#define    POLLOUT   0x0004    /* file descriptor is writeable */

```

# epoll