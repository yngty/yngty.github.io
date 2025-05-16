---
title: 深入理解 HKDF：HMAC 密钥派生函数
date: 2025-04-18 10:33:24
tags:
- HKDF
- HMAC
categories:
- Cryptographic Algorithms
mathjax: true
---

# 介绍
在加密领域，密钥派生函数（`KDF`）是通过从一个初始密钥（通常称为“种子密钥”或“主密钥”）生成多个密钥的算法。`HKDF（HMAC-based Key Derivation Function）`是一种基于 `HMAC` 的密钥派生函数，它被设计用于从一个或多个输入密钥材料中生成多个安全的输出密钥。`HKDF` 是一个简洁且具有高度安全性的 `KDF`，广泛用于生成加密协议中的密钥（如 `TLS`、`IPSec` 等）。

在这篇文章中，我们将深入探讨 `HKDF` 的原理、计算过程以及应用场景，帮助你更好地理解这个关键的密码学工具。
<!--more-->

# HKDF 的工作原理

`HKDF` 是一种伪随机函数（`PRF`），其核心思想是从一个初始的种子密钥（或称主密钥）中生成多个密钥，并且能够提供高度的安全性。`HKDF` 通过两步过程实现密钥派生：**提取（Extract）** 和 **扩展（Expand）**。

## 1. 提取（Extract）

提取过程是将输入的密钥材料（通常是随机的）和一个盐（`salt`）进行处理，得到一个固定大小的伪随机输出。盐的作用是增加随机性，避免相同的输入产生相同的输出，从而增加安全性。

- **输入**：
    - `IKM（Input Key Material）`：初始密钥材料，通常是一个主密钥或其他随机数据。
    - `salt`（盐）：一个可选的随机值，通常是由一个非秘密值（例如，固定值或随机生成的值）组成。盐可以为空，**但为空时应使用特定的默认值（如全零，长度则为所采用哈希函数的散列值长度）**。

- **输出**：
    - `PRK（Pseudorandom Key）`：一个固定长度的伪随机密钥（`PRK`），通常使用 `HMAC` 作为提取过程的核心操作。

`HMAC` 算法将在提取过程中发挥作用，用盐和输入密钥材料（`IKM`）生成伪随机密钥（`PRK`）。

$$
\begin{aligned}

&\mathrm{PRK} = \mathrm{HMAC}(\text{salt},\ \text{IKM}) \\
\end{aligned}
$$


## 2. 扩展（Expand）

扩展过程使用从提取步骤得到的伪随机密钥（`PRK`）和一些额外的参数（如输出的密钥长度和上下文信息），进一步生成所需数量的密钥。扩展步骤是通过连续应用 `HMAC` 来逐步生成所需的密钥。

- **输入**：

    - `PRK`（伪随机密钥）：提取过程的输出。
    - `info`（上下文信息）：可选的额外数据（如协议标识符、会话信息等），用于区分不同的密钥生成需求。
    - `L`（所需密钥长度）：所需的输出密钥长度，单位是字节 $L \leq 255 \times \text{HashLen}$


- **输出**：

    - `OKM` 输出密钥材料，长度为 `L` 字节

扩展阶段通过反复调用 `HMAC` 来生成一个或多个块，每个块的大小为 `HMAC` 输出长度（`HashLen`），直到输出满足总长度 `L`。公式如下：


$$
\begin{aligned}
& T_0 = \text{空字符串（zero-length）} \\\\
& T_1 = \mathrm{HMAC}(PRK,\ T_0\ \texttt{||}\ \text{info}\ \texttt{||}\ \texttt{0x01}) \\\\
& T_2 = \mathrm{HMAC}(PRK,\ T_1\ \texttt{||}\ \text{info}\ \texttt{||}\ \texttt{0x02}) \\\\
& T_3 = \mathrm{HMAC}(PRK,\ T_2\ \texttt{||}\ \text{info}\ \texttt{||}\ \texttt{0x03}) \\\\
& \vdots \\\\
& OKM = T_1\ \texttt{||}\ T_2\ \texttt{||}\ T_3\ \cdots \quad (\text{直到达到所需长度 L})
\end{aligned}
$$


# HKDF 的详细计算过程

我们以 使用 **SHA-256（HashLen = 32 字节）且需要输出 64 字节密钥** 为例。


## 步骤 1：提取（Extract）
首先，我们使用 `HMAC` 对 `IKM` 和 `salt` 进行计算，得到伪随机密钥（`PRK`）。

$$
\mathrm{PRK} = \mathrm{HMAC}(\text{salt},\ \text{IKM})
$$

`HMAC` 会根据 `SHA-256` 哈希函数对 `salt || IKM` 进行计算，得到固定长度的输出，即伪随机密钥（`PRK`）。

## 步骤 2：扩展（Expand）
接下来，使用 `PRK` 和 `info` 来生成最终的密钥。在这个过程中，我们会多次使用 `HMAC` 来生成所需的密钥。

###  1. 初始化：
- $T_0$ = ""（空字符串）
- `PRK` = 来自提取阶段的 `HMAC` 输出（`32` 字节）
- `info` = 可选上下文信息（比如 b"context info"）



### 2. 计算第一个区块 $T_1$

$$
T_1 = \mathrm{HMAC}(\mathrm{PRK},\ T_0\ \texttt{||}\ \text{info}\ \texttt{||}\ \texttt{0x01})
$$

- $T_0$ `= ""`

- 拼接内容为：`info + 0x01`

- 调用 `HMAC: HMAC(PRK, info + 0x01)`

- 输出 $T_1$（`32` 字节）


### 3. 计算第二个区块 $T_2$

$$
T_2 = \mathrm{HMAC}(\mathrm{PRK},\ T_1\ \texttt{||}\ \text{info}\ \texttt{||}\ \texttt{0x02})
$$


- 拼接内容为：$T_1$ + info + 0x02

- 调用 `HMAC`: HMAC(PRK, $T_1$ + info + 0x02)

- 输出 $T_2$（`32` 字节）



### 4. 拼接输出

$$
\mathrm{OKM} = T_1\ \texttt{||}\ T_2 = 64\ \text{字节}
$$

如果需要更多字节（如 `80` 字节），则继续生成 $T_3$：

$$
T_3 = \mathrm{HMAC}(\mathrm{PRK},\ T_2\ \texttt{||}\ \text{info}\ \texttt{||}\ \texttt{0x03})
$$

依此类推，直到拼接够 `L` 字节为止。


# HKDF 的安全性

`HKDF` 的安全性依赖于 `HMAC` 的安全性和良好的输入选择。由于 `HMAC` 基于强大的哈希函数（如 `SHA-256`），并且能够有效地防止碰撞攻击、重放攻击等，它本身是非常安全的。使用高质量的盐和上下文信息（`info`）可以进一步增加安全性，防止生成相同的密钥。

## 盐的作用

盐（`salt`）的作用是防止相同的输入材料（`IKM`）生成相同的伪随机密钥（`PRK`）。如果不同的密钥材料没有使用盐，可能会导致同样的密钥材料每次生成相同的派生密钥，降低安全性。因此，使用随机或不可预测的盐是非常重要的。

## info 参数
`info` 参数的作用是提供额外的上下文信息，使得即使相同的主密钥和盐被用于生成不同的密钥，也能确保它们的差异性。例如，在 `TLS` 中，可以将会话ID作为 `info` 参数，保证不同会话中生成的密钥不同。

# HKDF 的应用场景

`HKDF` 在许多加密协议和应用中都有广泛的应用。以下是一些典型场景：

`TLS/SSL`：在 `TLS` 连接中，`HKDF` 用于派生会话密钥，保证每次连接都有不同的密钥，增加安全性。

`IPSec`：用于加密和认证的密钥派生。

密码学协议：例如，密钥交换协议中，`HKDF` 用于从共享密钥材料中派生密钥。

`API` 密钥生成：从主密钥生成多个 `API` 密钥，以便对不同的应用进行认证。

# 总结

`HKDF` 是一种非常强大的密钥派生函数，它结合了 `HMAC` 和两步提取、扩展的过程，能够生成高度安全的密钥。通过从主密钥材料中提取出伪随机密钥，再通过扩展生成所需的多个密钥，`HKDF` 提供了一个灵活且可靠的密钥派生方案。无论是在加密协议、密钥交换还是 `API` 认证中，`HKDF` 都是一个非常有用的工具。

# 参考文献

- [HMAC-based Extract-and-Expand Key Derivation Function (HKDF)](https://datatracker.ietf.org/doc/html/rfc5869)
