---
title: HTTP 协议
date: 2023-07-17 11:42:24
tags:
- http
- 网络
- 网络协议
categories:
- Network
---

# HTTP 是什么？

`HTTP` 是超文本传输协议，也就是 `HyperText Transfer Protocol`。

# URI 的完整图解
```
  foo://example.com:8042/over/there?name=ferret#nose
  \_/   \______________/\_________/ \_________/ \__/
   |           |            |            |        |
scheme     authority       path        query   fragment
   |   _____________________|__
  / \ /                        \
  urn:example:animal:ferret:nose
```

# 消息格式

```
start-line
*( header-field CRLF )
CRLF
[ message-body ]
```
<!--more-->
## start-line

- request line
    - method SP request-target SP HTTP-version CRLF
- status line
    - HTTP-version SP status-code SP reason-phrase CRLF

##  Header Fields

- field-name ":" OWS field-value OWS

## Message Body

在一个请求中是否会出现消息体，以消息头中是否带有 `Content-Length` 或者 `Transfer-Encoding` 头字段作为信号


# Proxy 代理

## 普通代理

[RFC 7230 - HTTP/1.1: Message Syntax and Routing](https://datatracker.ietf.org/doc/html/rfc7230#section-2.3) 这种代理扮演的是「中间人」角色，对于连接到它的客户端来说，它是服务端；对于要连接的服务端来说，它是客户端。它就负责在两端之间来回传送 `HTTP` 报文。

## 隧道代理

[RFC 7231 - HTTP/1.1: Semantics and Content](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.6) `HTTP` 客户端通过 `CONNECT` 方法请求隧道代理创建一条到达任意目的服务器和端口的 `TCP` 连接，并对客户端和服务器之间的后继数据进行盲转发。

```
CONNECT server.example.com:80 HTTP/1.1
```


# `HTTP/1.1、HTTP/2、HTTP/3` 演变

## `HTTP/1.1` 的优化

- 长连接
- 支持管道 （`pipeline`）网络传输

但 `HTTP/1.1` 还是有性能瓶颈：

- 请求 / 响应头部（`Header`）未经压缩就发送，首部信息越多延迟越大。只能压缩 `Body` 的部分；
- 发送冗长的首部。每次互相发送相同的首部造成的浪费较多；
- 服务器是按请求的顺序响应的，如果服务器响应慢，会招致客户端一直请求不到数据，也就是队头阻塞；
- 没有请求优先级控制；
- 请求只能从客户端开始，服务器只能被动响应。

## `HTTP/2` 的优化

- 头部压缩
- 二进制格式
- 并发传输
- 服务器主动推送资源

## `HTTP/3` 的优化

`HTTP/3` 基于 `UDP` 的 `QUIC` 协议 可以实现类似 `TCP` 的可靠性传输, 有以下 `3` 个特点。

- 无队头阻塞
- 更快的连接建立
- 连接迁移
