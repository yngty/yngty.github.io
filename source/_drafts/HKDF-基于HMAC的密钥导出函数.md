---
title: HKDF(基于HMAC的密钥导出函数)
tags:
tags:
- SSL/TLS
- 密码学
---

# 什么是HKDF

`KDF` 是密码学系统中必要的组件。它的目的是把一个 `key` 拓展成多个从密码学角度来上说是安全的key。

`HKDF` 是 `HMAC-based Extract-and-Expand Key Derivation Function` 的缩写,意为基于 `HMAC` 的提取和扩展密钥派生函数。

`HKDF` 的主要目的使用原始的密钥材料,派生出一个或更多个能达到密码学强度的密钥(主要是保证随机性)。

`HKDF` 包含两个基本模块,或者说两个基本使用步骤:
- 提取 `Extract`
- 扩展 `Expand`

[RFC5869](https://datatracker.ietf.org/doc/html/rfc5869)

