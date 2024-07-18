---
title: TCP-TFO快速打开
date: 2024-03-21 11:25:45
tags:
- tcp
- tfo
categories:
- Network
---

# 简介

`TCP` 快速打开（`TCP Fast Open`，简称`TFO`）是对计算机网络中传输控制协议（`TCP`）连接的一种简化握手手续的拓展，用于提高两端点间连接的打开速度。

它通过握手开始时的 `SYN` 包中的 `TFO cookie`（一个 `TCP` 选项）来验证一个之前连接过的客户端。如果验证成功，它可以在三次握手最终的 `ACK` 包收到之前就开始发送数据，这样便跳过了一个绕路的行为，更在传输开始时就降低了延迟。这个加密的 `Cookie` 被存储在客户端，在一开始的连接时被设定好。然后每当客户端连接时，这个 `Cookie` 被重复返回。

> **最显著的优点是可以利用握手去除一个往返 RTT**

# 开启 TFO

`net.ipv4.tcp_fastopen` 是 `Linux` 内核中的一个配置参数，它用于控制 `TCP Fast Open` 功能。
具体地，`net.ipv4.tcp_fastopen` 的值可以是以下几种：

- `0`：禁用 `TCP Fast Open` 功能。
- `1`：在客户端启用 `TCP Fast Open` 功能。
- `2`：在服务器端启用 `TCP Fast Open` 功能。
- `3`：在客户端和服务器端都启用 `TCP Fast Open` 功能。

通过设置这个参数，可以根据实际需求选择是否启用和在哪一端启用 `TCP Fast Open`，从而优化网络性能。
<!--more-->

# 请求 `Fast Open Cookie` 的过程如下：

- 客户端发送一个 `SYN` 包，头部包含 `Fast Open` 选项，且该选项的 `Cookie` 为空，这表明客户端请求 `Fast Open Cookie`
- 服务端收取 `SYN` 包以后，生成一个 `cookie` 值（一串字符串）
- 服务端发送 `SYN` + `ACK` 包，在 `Options` 的 `Fast Open` 选项中设置 `cookie` 的值
- 客户端缓存服务端的 `IP` 和收到的 `cookie` 值

第一次过后，客户端就有了缓存在本地的 `cookie` 值，后面的握手和数据传输过程如下：

- 客户端发送 `SYN` 数据包，里面包含数据和之前缓存在本地的 `Fast Open Cookie`。
- 服务端检验收到的 `TFO Cookie` 和传输的数据是否合法。
    - 如果合法就会返回 `SYN` + `ACK` 包进行确认并将数据包传递给应用层
    - 如果不合法就会丢弃数据包，走正常三次握手流程（只会确认 `SYN`）
- 服务端程序收到数据以后可以握手完成之前发送响应数据给客户端了
- 客户端发送 `ACK` 包，确认第二步的 `SYN` 包和数据（如果有的话）
后面的过程就跟非 `TFO` 连接过程一样了

我们看看 `curl` 如何支持 `fast open`, 通过 `strace` 抓下:

```shell
strace curl --tcp-fastopen http://192.168.14.168
```
![tfo_curl](../../images/tfo_curl.jpg)

我们看下在服务器的抓包信息:
## 第一次请求
`fast open cookie` 为空

## 第二次请求
第一个 `SYN` 包被识别成了 `HTTP` 请求
![tfo](../../images/tfo_2.jpg)

![tfo_1](../../images/tfo_2_1.png)
![tfo_2](../../images/tfo_2_2.png)

# 参考资料

- [TCP快速打开 WIKI](https://zh.wikipedia.org/wiki/TCP%E5%BF%AB%E9%80%9F%E6%89%93%E5%BC%80)
