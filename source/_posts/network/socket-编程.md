---
title: socket 编程
date: 2023-07-24 10:38:44
tags:
- tcp
- socket
categories:
- Network
---


# 1. socket地址结构

##  `sockaddr_in`  

`ipv4` 协议的地址结构是 `sockaddr_in`，`ipv6` 的地址结构是`sockaddr_in6`。

```c
#include <arpa/inet.h>
//#include<netinet/in.h>

struct sockaddr_in
{
    sa_family_t     sin_family;
    in_port_t       sin_port;	    /* Port number.  */
    struct in_addr  sin_addr;		/* Internet address.  */

    /* Pad to size of `struct sockaddr'.  */
    unsigned char sin_zero[sizeof (struct sockaddr) -
            __SOCKADDR_COMMON_SIZE -
            sizeof (in_port_t) -
            sizeof (struct in_addr)];
};
```

- `sin_family`：表示地址簇，`ipv4: AF_INET, ipv6: AF_INET6`，
- `sin_port`：16位的端口号
- `sin_addr`：点分十进制。

## 通用地址结构  
    
结构体是 `sockaddr`，方便可以接受 `ipv4/ipv6` 的地址结构。之所以采用 `sockaddr`，而不采用 `void*`  是因为 `BSD` 设计套接字的时候大约是 `1982` 年，那个时候的 `C` 语言还没有`void *` 的支持。

```c
struct sockaddr {
    sa_family_t  sa_family;
    char         ss_data[14];
};  
```
<!--more-->
将 `sockaddr_in/sockaddr_in6` 强制转换为 `sockaddr` ，通过 `sockaddr` 的 `sa_family` 来分别使用的是 `ipv4/ipv6`。

## 网络字节序列和主机字节序列转换  
`TCP/IP` 协议规定，网络传输字节按照**大端字节序列**方式      

- 大端：低地址存储在高位。
- 小端：低地址存低位。  

### `sin_port` 转换

```c
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);  // host to net long(32位置)

uint16_t htons(uint16_t hostshort); // host to net short(16位置)

uint32_t ntohl(uint32_t netlong);   // net to host long

uint16_t ntohs(uint16_t netshort);
```

### `sin_addr` 转换  

与协议无关的的转换函数，即 `ipv4/ipv6`都可以。
```c
#include <arpa/inet.h>

int inet_pton(int af, const char *src, void *dst);

const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
```
`ipv4` 专用的转换函数
```c
#include <arpa/inet.h>

int 
inet_aton(const char *cp, struct in_addr *inp);

char 
*inet_ntoa(struct in_addr in);

in_addr_t 
inet_addr(const char *cp); // 有风险，不建议使用
```
# 2. socket 函数  

## `socket`
```c
#include <sys/types.h>         
#include <sys/socket.h>

int socket(int family, int type, int protocol);
```
- 返回值 
    - 失败时返回 `-1`，成功是返回一个**非零整数值**，表示套接字描述符，`sockfd`。
- 参数
    - `family`：
        - `PF_INET`
        - `PF_INET6`
        - `PF_LOCAL`
    - `type`：
        - `SOCK_STREAM` : 表示的是字节流，对应 `TCP`；
        - `SOCK_DGRAM` : 表示的是数据报，对应 `UDP`；
        - `SOCK_RAW` : 表示的是原始套接字。
        - 还可和 `SOCK_NOBLOCK` 和`SOCK_CLOEXEC` 进行组合使用。
    - `protocol`：
        - `0`，原本是用来指定通信协议的，但现在基本废弃。因为协议已经通过前面两个参数指定完成。目前一般写成 `0` 即可。
- 状态：  
    - 创建 `sockfd` 以后，处于 `CLOSED` 状态。

## `connect`
```c
#include <sys/types.h>         
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```
客户端调用函数。  

- 返回值
    - 成功返回 `0`，失败返回 `-1`。
    - 客户端调用 `connect` 函数，**会激发`TCP`的三次握手过程**，而且仅仅在连接成功或者失败才返回。**客户端是在第二个分节返回，服务端是第三个分节返回**。  
    - *`ETIMEOUT`*：若客户端没有收到 `SYN` 分节响应，就会返回这个错误。
    - *`ECONNREFUSED`*：若对客户端的 `SYN` 分节响应的是 `RST`，表示服务器主机在指定的端口上没有进程与之连接，客户端一接受到 `RST` 就返回 `ECONNREFUSED` 错误。
    - 不可达错误。
- 参数
    - `sockfd`：是`socket`函数返回值。
    - `sockaddr`：是套接字的地址结构，`sockaddr_int/sockaddr_in6` 强制转换而来。
    - `addrlen`：传入的地址结构大小。  
- 状态：  
    `connect` 会使得当前套接字从 `closed` 状态转移到 `SYN_SENT` 状态，如成功再转移到`ESTABLISHED` 状态，若失败则该套接字不可用，**必须关闭**。

## `bind`

```c
#include <sys/types.h>         
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```
服务器端调用函数。

- 参数
    - `addr`
        - 可以使用`通配地址` 对于 `IPv4` 的地址来说，使用 `INADDR_ANY` 来完成通配地址的设置；对于 `IPv6` 的地址来说，使用 `IN6ADDR_ANY` 来完成通配地址的设置
        - `0`, 系统随机分配
    -  `port`
        - 绑定端口，一般选择大于 `1024` 的端口。
        - `0`, 系统随机分配


```c
struct sockaddr_in name;
name.sin_addr.s_addr = htonl (INADDR_ANY); /* IPV4 通配地址 */
```
## `listen`

```c
#include <sys/types.h>         
#include <sys/socket.h>

int listen(int sockfd, int backlog);
```
`listen` 函数由**服务器端**调用。

初始化创建的套接字，可以认为是一个"主动"套接字，其目的是之后主动发起请求（通过调用 `connect` 函数）。通过 `listen` 函数，可以将原来的"主动"套接字转换为"被动"套接字，告诉操作系统内核："我这个套接字是用来等待用户请求的。", 操作系统内核会为此做好接收用户请求的一切准备，比如完成连接队列。

- 返回值 
    - 失败时返回 `-1`，成功返回 `0`。
- 参数
    - `sockfd`
        - 初始化套接字
    - `backlog`
        - 内核为相应套接字排队的最大连接个数，这个参数的大小决定了可以接收的并发数目。
        - 内核为每个**监听**套接字维护两个队列：  
            (1) 半连接队列（ `SYN` 队列）：接收到一个 `SYN` 建立连接请求，处于 `SYN_RCVD` 状态；<br>
            (2) 全连接队列（ `Accept` 队列）：已完成 `TCP` 三次握手过程，处于 `ESTABLISHED` 状态； 
            
            在早期 `Linux` 内核 `backlog` 是 `SYN` 队列大小，也就是未完成的队列大小。在 `Linux` 内核 `2.2` 之后，`SYN` 队列由 `/proc/sys/net/ipv4/tcp_max_syn_backlog`指定； `backlog` 变成 `Accept` 队列，也就是已完成连接建立的队列长度，所以现在通常认为 `backlog` 是 `accept` 队列。但是上限值是内核参数 `somaxconn` 的大小，也就说 ** `Accept` 队列长度 = `min(backlog, somaxconn)`**。


- 状态转移
    - 当来自客户的 `SYN` 分节到达时，`TCP` 在未连接队列创建一个新项，然后响应以三次握手的第二个分节，这一项一直保留在未完成连接队列中，直到三次握手的第三个分节到达或者超时。如果到达，该项就从未完成连接队列中移到已完成连接队列的队尾。当调用 `accept` 时，已完成连接队列的队首将作为 `accept` 的返回值，如果已完成连接队列是空，那么调用 `accept` 函数的进程会进入睡眠状态。

## `accept`

```c
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```
调用 `accept` 时，已完成连接队列的队首将作为 `accept` 的返回值，如果已完成连接队列是空，那么调用进程进入睡眠状态。成功返回客户端的已连接套接字`connfd`，失败返回 `-1`。

`accpet` 函数返回时，表示已连接套接字 `connfd` 和服务器端的监听套接字 `listenfd` 完成了三次握手。

- *`EMFILE`*  
    如果函数`accept`返回`EMFILE`，即文件描述符过多，怎么处理？

    >先实现准备一个空闲的文件描述符 *`/dev/null`*。遇到这种情况，先关闭这个空闲的文件描述符，就可以获得一个文件描述名额，然后再`accept`就可以拿到这个连接的`socket`文件描述符，随后立即`close`，就优雅的断开了与客户端的连接，最后重新打开空闲文件，以备这种情况再次出现。

## `close`

```c
#include <unistd.h>

int close(int fd);
```

这个函数表示的把该套接字标记为已关闭，然后立即返回到调用进程，该套接字描述符不能再被调用进程使用。  
- 注意事项
    - 由于描述符是引用计数，`close` 只是减少该引用计数，只有当该引用计数为 `0` 时才会引用终止序列。
    - tcp会先将已经排队等待发送到对端的任何数据发送过去，然后再发送终止序列`FIN`。因此，调用 `close` 不是立即发送终止序列。

## `shutdown`

```c
#include <sys/socket.h>

int shutdown(int sockfd, int how);
```

`shutdown` 解决的是 `close` 的两个限制：
- `close` 把描述符计数减一，仅仅在计数变为 `0` 时才关闭套接字。`shutdown` 可以不管描述符计数就激发 `TCP` 的正常连接终止序列。
- `close` 终止**读和写**两个方向的数据传递，`shutdown` 是半关闭，可以只是关闭一个方向数据流。

- 参数
    - `how`:
        - `SHUT_RD`：关闭读
        - `SHUT_WR`：关闭写
        - `SHUT_RDWR`：关闭读写