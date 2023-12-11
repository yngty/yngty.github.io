---
title: Application-Layer Protocol Negotiation
tags:
  - SSL/TLS
date: 2023-12-11 10:07:03
---

# Introduction

`ALPN`(`Application-Layer Protocol Negotiation`)应用层协议协商, 当单个服务器端口号（例如端口 `443`）上支持多个应用程序协议时，客户端和服务器需要协商用于每个连接的应用程序协议。希望在不增加客户端和服务器之间的网络往返次数的情况下完成此协商，因为每次往返都会降低最终用户的体验。

`ALPN` 作为 `TSL`的扩展，客户端会将支持的应用程序协议列表作为 `TLS ClientHello` 消息的一部分发送给服务器，服务器选择一个协议，并将所选协议作为 `TLS ServerHello` 消息的一部分发送给客户端。因此，可以在 `TLS` 握手中完成应用协议协商，而无需添加网络往返，并且允许服务器根据需要，将不同的证书与每个应用协议相关联。

<!--more-->
通过 `OpenSSL` 命令行工具，快速查看 `HTTP/2` 服务是否支持 `ALPN` 扩展：

```shell
openssl s_client -alpn h2 -servername ipinfo.io -connect ipinfo.io:443 < /dev/null | grep ALPN 
```
- 提示 `unknown option -alpn`:  `OpenSSL` 版本太低, `OpenSSL 1.0.2` 才开始支持 `ALPN` 需要升级高版本
- 结果包含 `ALPN protocol: h2`，说明服务端支持 `ALPN`
- 结果包含 `No ALPN negotiated`，说明服务端不支持 `ALPN`

# `Application-Layer Protocol Negotiation`

## `The Application-Layer Protocol Negotiation Extension`

定义新的扩展类型（`application_layer_protocol_negotiation（16`），并且可以由客户端包括在其 `ClientHello` 消息中。
```c
enum {
    application_layer_protocol_negotiation(16), (65535)
} ExtensionType;
```

（`application_layer_protocol_negotiation（16`）扩展的 `extension_data` 字段应包含`ProtocolNameList` 值。

```c
opaque ProtocolName<1..2^8-1>;

struct {
    ProtocolName protocol_name_list<2..2^16-1>
} ProtocolNameList;
```

`ProtocolNameList` 按优先级从高到低包含客户端发布的协议列表。 协议是由 `IANA` 注册的不透明非空字节串命名的。不能包含空字符串，并且不能截断字节字符串。

接收到包含 `application_layer_protocol_negotiation` 扩展名的 `ClientHello` 的服务器可以向客户端返回合适的协议选择作为响应。服务器将忽略它无法识别的任何协议名称。一个新的 `ServerHello` 扩展类型(`application_layer_protocol_negotiation(16)`) 可以在 `ServerHello` 消息扩展中返回给客户端。(`application_layer_protocol_negotiation(16)`) 扩展名的 `extension_data` 字段的结构与上述针对客户端 `extension_data` 的描述相同，只是 `ProtocolNameList` 必须包含一个 `ProtocolName`。


```
   Client                                              Server
   ClientHello                     -------->       ServerHello
     (ALPN extension &                               (ALPN extension &
      list of protocols)                              selected protocol)
                                                   Certificate*
                                                   ServerKeyExchange*
                                                   CertificateRequest*
                                   <--------       ServerHelloDone
   Certificate*
   ClientKeyExchange
   CertificateVerify*
   [ChangeCipherSpec]
   Finished                        -------->
                                                   [ChangeCipherSpec]
                                   <--------       Finished
   Application Data                <------->       Application Data
```

##  Protocol Selection

期望服务器将具有优先级支持的协议列表，并且仅在客户端支持的情况下才选择协议。在这种情况下，服务器应该选择它所支持的，并且也是由客户端发布的最优先的协议。如果服务器不支持客户端传过来的协议，则服务器应以 `"no_application_protocol"` `alert` 错误回应。

```c
enum {
    no_application_protocol(120),
    (255)
} AlertDescription;
```

## `Wireshark` 抓包

在 `TLS1.3` 中:

- `ClientHello`消息:
![ClientHello](/images/alpn_client_hello_tls1_3.png)

- `ServerHello`消息:
![ServerHello](/images/alpn_server_hello_tls1_3.png)

# 参考资料

* [Application-Layer Protocol Negotiation Extension](https://datatracker.ietf.org/doc/html/rfc7301)