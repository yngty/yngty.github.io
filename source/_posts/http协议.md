---
title: http协议
date: 2023-07-17 11:42:24
tags: HTTP
---


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

