---
title: MTU与MSS的奥秘
date: 2024-03-18 14:44:24
tags:
- tcp
- MTU
- MSS
categories:
- Network
---
# 最大传输单元（Maximum Transmission Unit, MTU）

数据链路层传输的帧大小是有限制的，不能把一个太大的包直接塞给链路层，这个限制被称为「最大传输单元（`Maximum Transmission Unit, MTU`）」


不同的数据链路层的 `MTU` 是不同的。通过 `netstat -i` 可以查看网卡的 mtu，比如在 我的 `ubuntu` 机器上可以看到

```shell
-> netstat -i
Kernel Interface table
Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
ens33     1500     4513      0      0 0          2872      0      0      0 BMRU
lo       65536     5307      0      0 0          5307      0      0      0 LRU
```

# IP 分段

# 网络中的木桶效应：路径 MTU

# TCP 最大段大小（Max Segment Size，MSS）

# 为什么有时候抓包看到的单个数据包大于 MTU

# TCP 套接字选项 TCP_MAXSEG

# 小结

IP 数据包长度在超过链路的 MTU 时在发送之前需要分片，而 TCP 层为了 IP 层不用分片主动将包切割成 MSS 大小。
