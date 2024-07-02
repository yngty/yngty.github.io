---
title: 如何正确使用TCP
date: 2023-07-24 15:12:44
tags:
- tcp
categories:
- Network
---

# `SO_REUSEADDR`

- `TCP` 服务器能够在杀掉或崩溃后快速重启
- 也适用 `fork-per-connection` 服务器模型。

```c++
int optval = 1;
setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(optval));
```
# 忽略 `SIGPIPE`

程序向对方已经关闭的管道，写数据，会收到 `SIGPIPE` 信号。`write` 系统调用返回 `-1`收到 `errono EPIPE`。 `SIGPIPE` 信号默认行为终止进程。我们应该忽略 `SIGPIPE` 信号。

```c++
if (signal(SIGPIPE, SIG_IGN) == SIG_ERR) {
    return 1;
}
```

<!--more-->
# TCP_NODELAY

`Nagle` 算法避免发送大量的小包，防止小包泛滥于网络，理想情况下，对于一个 `TCP` 连接而言，只允许一个未被 `ACK`的包存在于网络。

`Nagle` 算法规则:
- 如果包长度达到 `MSS`，则允许发送
- 如果包含 `FIN`，则允许发送
- 如果设置了 `TCP_NODELAY`，则允许发送
- 未设置 `TCP_CORK` 选项时，若所有发出去的小数据包（包长度小于`MSS`）均被确认，则允许发送
- 上述条件都未满足，但发生了超时（一般为`200ms`），则立即发送。

通过设置 `TCP_NODELAY` 禁用 `nagle` 算法。

```c++
int flag = 1;
setsockopt(sockfd, IPPROTO_TCP, TCP_NODELAY, (char*)&flag, sizeof(int));
```

# 正确关闭 `TCP` 连接

如果协议栈接受缓存区中有数据，程序还没有读，直接调用 `close` 函数，`TCP` 协议栈会发送 `RST` 包，强行断开连接，如果协议栈发送缓存区还有有数据，则对方没能收到，造成数据丢失。 对于无格式的协议正确的做法是：

```
发送方: send() + shutdown(WR) + read()->0 + close()
接收方: read()->0 + 没有数据发送 + close()
```

如果遇到恶意或者是有 `bug` 的 `client`，一直不 `close`，发送方一直阻塞在 `read` , 建议加一个超时机制退出程序，这是为了程序安全 `security`，不是为了数据安全 `safety` 完整性 `Integrity`。

依赖 `shutdown write` 会发送 `FIN`，`end of file`，更好的办法是设计协议，把数据长度包含进来，接收方可以主动判断数据是否收全。

# `Socket` 选项之 `SO_LINGER`
