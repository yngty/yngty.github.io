---
title: 深入理解 iptables：Linux 网络包过滤的核心机制
date: 2025-05-26 10:00:00
tags:
  - Linux
  - iptables
  - Netfilter
  - 防火墙
  - 网络安全
categories:
  - Network
  - 网络安全
  - Linux 内核
---

## 一、前言

在 `Linux` 网络安全与数据包处理领域，`iptables` 是不可忽视的重要工具。它是 `Linux` 系统上用于配置防火墙规则的用户空间工具，底层依赖内核模块 Netfilter 实现数据包的捕获与处理。

---

<!--more-->

## 二、Netfilter 与 iptables 的关系

- **Netfilter** 是 `Linux` 内核中网络栈的一部分，提供钩子（`hook`）接口来允许数据包在经过协议栈的不同阶段时被拦截。
- **iptables** 是用户空间的控制工具，通过 `libiptc` 库与内核交互，配置规则链表。

```bash
iptables → libiptc → Netfilter 内核模块 → 网络数据包处理
```

## 三、iptables 架构解析

### 3.1 表（Tables）

`iptables` 使用多个**表**来区分不同的用途。每张表包含一组预定义的“链”（`Chain`），你可以在链上添加规则（`Rule`）。

| 表名       | 功能说明             | 常用链                             | 应用场景示例                     |
|------------|----------------------|------------------------------------|----------------------------------|
| `filter`   | 默认表，数据包过滤   | `INPUT`, `OUTPUT`, `FORWARD`       | 限制本机访问、端口放行           |
| `nat`      | 网络地址转换         | `PREROUTING`, `POSTROUTING`, `OUTPUT` | `DNAT`、`SNAT`、端口映射           |
| `mangle`   | 修改 `IP` 头部字段     | 所有链均可使用                     | 设置 `TOS`、`TTL`、打标记（mark）    |
| `raw`      | 跳过连接跟踪         | `PREROUTING`, `OUTPUT`             | 跳过 `conntrack`，提高性能         |
| `security` | 安全模块配合使用     | `INPUT`, `OUTPUT`, `FORWARD`       | `SELinux` 等 `LSM` 模块使用较少      |

> 💡 提示：如果你未指定 `-t` 参数，默认操作的是 `filter` 表。

---

### 3.2 链（Chains）

链定义了数据包在内核网络栈中的处理位置。每个表中的链负责特定阶段的数据包处理流程。

| 链名         | 描述                           | 常用于表       |
|--------------|--------------------------------|----------------|
| `PREROUTING` | 数据包进入内核前的第一个处理点 | `nat`, `mangle`, `raw` |
| `INPUT`      | 到达本机的入站数据包处理链     | `filter`, `mangle`, `security` |
| `FORWARD`    | 需要转发到其他主机的数据包     | 同上           |
| `OUTPUT`     | 本机产生的出站数据包处理链     | 所有表         |
| `POSTROUTING`| 即将离开内核、发送到网络前处理 | `nat`, `mangle` |

**链的处理顺序（以一张图表示）**：

```
           ---> PREROUTING --> (路由判断)
          /                           \
INTERNET                                FORWARD ---> POSTROUTING ---> 网络
          \                           /
           ---> INPUT        OUTPUT ---> POSTROUTING ---> 网络
```

---


### 3.3 匹配（Match）机制详解

`iptables` 使用 `Match` 模块来对数据包的各种属性进行匹配。你可以堆叠多个匹配条件，类似于 `if` 条件组合。

常见匹配条件：

| 匹配项                  | 示例                         | 含义说明                         |
|-------------------------|------------------------------|----------------------------------|
| `-p <protocol>`         | `-p tcp`, `-p udp`           | 协议匹配                         |
| `--dport <port>`        | `--dport 80`                 | 目的端口                         |
| `--sport <port>`        | `--sport 22`                 | 源端口                           |
| `-s <source IP>`        | `-s 192.168.1.0/24`          | 匹配源地址                       |
| `-d <destination IP>`   | `-d 10.0.0.1`                | 匹配目标地址                     |
| `-i <interface>`        | `-i eth0`                    | 入接口匹配                       |
| `-o <interface>`        | `-o eth1`                    | 出接口匹配                       |
| `-m state`              | `--state NEW,ESTABLISHED`    | 匹配连接状态（依赖 `conntrack`）   |
| `-m conntrack`          | `--ctstate RELATED`          | 更强大的连接跟踪匹配             |
| `-m mac`                | `--mac-source 00:11:22:...`  | 匹配 MAC 地址                    |
| `-m comment`            | `--comment "标记规则"`       | 给规则加注释                     |


>
> `-p protocol`，实际上 `iptables` 会隐式加载同名 `match` 模块
---

### 3.4 目标（Target）机制详解

当规则匹配成功后，`Target` 决定如何处理这个数据包。可以是终结型（处理完就停止）或非终结型（继续匹配下一条规则）。

| 目标值        | 说明                               |
|---------------|------------------------------------|
| `ACCEPT`      | 接受数据包                         |
| `DROP`        | 丢弃数据包，不回应对方             |
| `REJECT`      | 拒绝数据包，并发送拒绝响应         |
| `LOG`         | 日志记录，后续还会继续匹配         |
| `DNAT`        | 改变目的地址（常用于端口转发）     |
| `SNAT`        | 改变源地址（常用于伪装出口地址）   |
| `MASQUERADE`  | 类似 `SNAT`，自动使用出口 `IP`（如拨号）|
| `MARK`        | 给数据包打标记（用于策略路由）     |
| `RETURN`      | 返回上层链                         |
| `跳转用户链`  | 跳转到用户自定义链继续处理         |

---

### 3.5 表和链的关系总结图

| 表 / 链       | PREROUTING | INPUT | FORWARD | OUTPUT | POSTROUTING |
|---------------|------------|-------|---------|--------|-------------|
| `filter`      | ✘          | ✅     | ✅       | ✅      | ✘           |
| `nat`         | ✅          | ✘     | ✘       | ✅      | ✅           |
| `mangle`      | ✅          | ✅     | ✅       | ✅      | ✅           |
| `raw`         | ✅          | ✘     | ✘       | ✅      | ✘           |
| `security`    | ✘          | ✅     | ✅       | ✅      | ✘           |

✅ 表示该表支持该链；✘ 表示该链不适用于该表。

---

## 四、iptables 命令使用说明

### 4.1 添加规则

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

- `-A INPUT`：添加到 `INPUT` 链末尾
- `-p tcp`：匹配协议为 `TCP`
- `--dport 22`：目标端口 `22`（SSH）
- `-j ACCEPT`：目标为 `ACCEPT`（放行）

### 4.2 插入规则到链首

```bash
sudo iptables -I INPUT 1 -p icmp -j ACCEPT
```

- 将放行 `ICMP` 的规则插入 `INPUT` 链的第一条规则位置

### 4.3 删除规则

```bash
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT
```

> 完整规则必须和原始规则一模一样才能删除
或者：

```bash
sudo iptables -D INPUT <编号>
```

使用 `iptables -L --line-numbers` 查看编号

### 4.4 列出规则

```bash
sudo iptables -L -n -v
```

- `-L`：列出当前所有规则
- `-n`：不解析域名，加快输出
- `-v`：详细模式，显示数据包和字节计数

### 4.5 清空规则

```bash
sudo iptables -F
```

- 清空所有链上的规则（仅当前表）

### 4.6 保存与恢复

保存当前规则到文件：

```bash
sudo iptables-save > /etc/iptables.rules
```

恢复规则：

```bash
sudo iptables-restore < /etc/iptables.rules
```

---

如需持久生效，请结合你的发行版使用相应机制：

- Debian/Ubuntu: 使用 `netfilter-persistent`
- CentOS/RHEL: 编辑 `/etc/sysconfig/iptables`

---
