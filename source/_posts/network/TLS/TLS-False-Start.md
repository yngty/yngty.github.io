---
title: TLS False Start
tags:
  - SSL/TLS
date: 2023-12-11 11:39:47
---


# 什么是TLS False Start？

在 `TLS` 协商第二阶段，浏览器发送 `ChangeCipherSpec` 和 `Finished` 后，立即发送加密的应用层数据，而无需等待服务器端的确认。

# 如何启用TLS False Start？

- 需要支持 `NPN/ALPN`
- 服务器端配置支持前向安全(`Forward Secrecy`)