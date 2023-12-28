---
title: TLS1.3(四) Cryptographic Computations
tags:
- SSL/TLS
---

# `7. Cryptographic Computations`

TLS握手建立一个或多个输入密钥，这些密钥组合在一起以创建实际的工作密钥材料，如下所详述。密钥推导过程同时包含了输入密钥和握手记录。请注意，因为握手记录包括 `Hello` 消息中的随机值，即使使用相同的输入密钥（例如，为多个连接使用相同的PSK），任何给定的握手都将有不同的流量密钥。

## `7.1.  Key Schedule`

密钥推导过程使用了在 `HKDF`（[RFC5869](https://datatracker.ietf.org/doc/html/rfc5869)）中定义的 `HKDF-Extract` 和 `HKDF-Expand` 函数，以及下文定义的函数：

```c
HKDF-Expand-Label(Secret, Label, Context, Length) =
            HKDF-Expand(Secret, HkdfLabel, Length)

在 `HkdfLabel` 被指定为:

struct {
    uint16 length = Length;
    opaque label<7..255> = "tls13 " + Label;
    opaque context<0..255> = Context;
} HkdfLabel;

Derive-Secret(Secret, Label, Messages) =
    HKDF-Expand-Label(Secret, Label,
                        Transcript-Hash(Messages), Hash.length)
```

`Transcript-Hash` 和 `HKDF` 使用的哈希函数是密码套件的哈希算法。`Hash.length` 是其输出长度（以字节为单位）。`Messages` 是指示的握手消息的串联，包括握手消息类型和长度字段，但不包括记录层标头。请注意，在某些情况下，会将零长度的上下文（由`""`表示）传递给 `HKDF-Expand-Label`。本文档中指定的标签都是 `ASCII`字符串，并且不包括尾随的 `NUL` 字节。

注意：对于常见的哈希函数，任何超过`12`个字符的标签都需要进行额外的哈希函数迭代来计算。本规范中的标签都已选择适应此限制。


# 

https://eprint.iacr.org/2015/914.pdf