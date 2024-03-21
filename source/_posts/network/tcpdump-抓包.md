---
title: tcpdump 抓包
date: 2023-07-19 13:49:02
tags:
- tools
- network
categories:
- Network
---

`tcpdump` 命令行抓包工具。

```
tcpdump [ -AbdDefhHIJKlLnNOpqStuUvxX# ] [ -B buffer_size ]
        [ -c count ] [ --count ] [ -C file_size ]
        [ -E spi@ipaddr algo:secret,...  ]
        [ -F file ] [ -G rotate_seconds ] [ -i interface ]
        [ --immediate-mode ] [ -j tstamp_type ] [ -k (metadata_arg) ]
        [ -m module ]
        [ -M secret ] [ --number ] [ --print ]
        [ -Q packet-metadata-filter ] [ -Q in|out|inout ]
        [ -r file ] [ -s snaplen ] [ -T type ] [ --version ]
        [ -V file ] [ -w file ] [ -W filecount ] [ -y datalinktype ]
        [ -z postrotate-command ] [ -Z user ]
        [ --time-stamp-precision=tstamp_precision ]
        [ --micro ] [ --nano ]
        [ expression ]
```
需要 `root` 权限运行:

```shell
> sudo tcpdump -i any
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
15:01:28.921972 ens33 Out IP ept.55660 > 123.208.120.34.bc.googleusercontent.com.https: Flags [P.], seq 1706429942:1706429981, ack 1072465052, win 63000, length 39
15:01:28.922423 ens33 In  IP 123.208.120.34.bc.googleusercontent.com.https > ept.55660: Flags [.], ack 39, win 64240, length 0
15:01:28.971505 lo    In  IP localhost.33425 > localhost.domain: 48367+ [1au] PTR? 123.208.120.34.in-addr.arpa. (56)
15:01:28.971785 ens33 Out IP ept.50072 > _gateway.domain: 32506+ PTR? 123.208.120.34.in-addr.arpa. (45)
15:01:28.977216 ens33 In  IP _gateway.domain > ept.50072: 32506 1/0/0 PTR 123.208.120.34.bc.googleusercontent.com. (98)
15:01:28.977492 lo    In  IP localhost.domain > localhost.33425: 48367 1/0/1 PTR 123.208.120.34.bc.googleusercontent.com. (109)
15:01:28.977796 lo    In  IP localhost.45140 > localhost.domain: 38317+ [1au] PTR? 168.14.168.192.in-addr.arpa. (56)
15:01:28.978066 ens33 Out IP ept.50288 > _gateway.domain: 21988+ PTR? 168.14.168.192.in-addr.arpa. (45)
```

`-i` 表示指定哪一个网卡，`any` 表示任意。有哪些网卡可以用 `ifconfig` 来查看，在我的虚拟机上，`ifconfig` 输出结果如下

```shell
-> ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.14.168  netmask 255.255.255.0  broadcast 192.168.14.255
        inet6 fe80::a421:b6c8:4404:333d  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:cc:cd:fb  txqueuelen 1000  (Ethernet)
        RX packets 4548  bytes 5135525 (5.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2897  bytes 1689085 (1.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 5307  bytes 391240 (391.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5307  bytes 391240 (391.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
如果只想查看 `ens33` 网卡经过的数据包，就可以使用 `tcpdump -i ens33`来指定。

<!--more-->

# 过滤主机：`host` 选项

```shell
sudo tcpdump -i any host 10.211.55.2
```

# 过滤源地址、目标地址：`src`、`dst`
```shell
sudo tcpdump -i any src 10.211.55.10
```
# 过滤端口：`port` 选项

```shell
sudo tcpdump -i any port 80
```

# 过滤指定端口范围内的流量

抓取 `21` 到 `23` 区间所有端口的流量

```shell
tcpdump portrange 21-23
```

# 禁用主机与端口解析：`-n` 与 `-nn` 选项

# 过滤协议

# 用 `ASCII` 格式查看包体内容：`-A` 选项

与 `-A` 对应的还有一个 `-X` 命令，用来同时用 `HEX` 和 `ASCII` 显示报文内容。

# 限制包大小：`-s` 选项

当包体很大，可以用 `-s` 选项截取部分报文内容，一般都跟 `-A` 一起使用。查看每个包体前 `500` 字节可以用下面的命令

```shell
sudo tcpdump -i any -nn port 80 -A -s 500
```

如果想显示包体所有内容，可以加上 `-s 0`


# 只抓取 `5` 个报文： `-c` 选项

使用 `-c number` 命令可以抓取 `number` 个报文后退出。在网络包交互非常频繁的服务器上抓包比较有用，可能运维人员只想抓取 `1000` 个包来分析一些网络问题，就比较有用了。

```shell
sudo tcpdump -i any -nn port 80  -c 5
```
# 数据报文输出到文件：`-w` 选项

`-w` 选项用来把数据报文输出到文件，比如下面的命令就是把所有 `80` 端口的数据输出到文件

```shell
sudo tcpdump -i any port 80 -w test.pcap
```
生成的 `pcap` 文件就可以用 `wireshark` 打开进行更详细的分析了

也可以加上 `-U` 强制立即写到本地磁盘，性能稍差


# 显示绝对的序号：`-S` 选项
默认情况下，`tcpdump` 显示的是从 `0` 开始的相对序号。如果想查看真正的绝对序号，可以用 `-S` 选项。

没有 `-S` 时的输出，`seq` 和 `ACK` 都是从 `0` 开始

# 高级技巧

`tcpdump` 真正强大的是可以用布尔运算符`and`（或`&&`）、`or`（或`||`）、`not`（或`!`）来组合出任意复杂的过滤器

抓取 `ip` 为 `10.211.55.10` 到端口 `3306` 的数据包

```shell
sudo tcpdump -i any host 10.211.55.10 and dst port 3306
```

抓取源 `ip` 为 `10.211.55.10`，目标端口除了 `22` 以外所有的流量

```shell
sudo tcpdump -i any src 10.211.55.10 and not dst port 22
```
## 复杂的分组

如果要抓取：来源 `ip` 为 `10.211.55.10` 且目标端口为 `3306` 或 `6379` 的包，按照前面的描述，我们会写出下面的语句

```shell
sudo tcpdump -i any src 10.211.55.10 and (dst port 3306 or 6379)
```
如果运行一下，就会发现执行报错了，因为包含了特殊字符 `()`，解决的办法是用单引号把复杂的组合条件包起来。

```shell
sudo tcpdump -i any 'src 10.211.55.10 and (dst port 3306 or 6379)'
```

如果想显示所有的 `RST` 包，要如何来写 `tcpdump` 的语句呢？先来说答案

```shell
tcpdump 'tcp[13] & 4 != 0'
```

要弄懂这个语句，必须要清楚 `TCP` 首部中 `offset` 为 `13` 的字节的第 `3` 比特位就是 `RST`

`tcp[13]` 表示 `tcp` 头部中偏移量为 `13` 字节, `!=0` 表示当前 `bit` 置 `1`，即存在此标记位，跟 `4` 做与运算是因为 `RST` 在 `TCP` 的标记位的位置在第 `3` 位(`00000100`)

如果想过滤 `SYN + ACK` 包，那就是 `SYN` 和 `ACK` 包同时置位（`00010010`），写成 `tcpdump` 语句就是
```shell
tcpdump 'tcp[13] & 18 != 0'
```
# 参考资料

[深入理解TCP协议-从原理到实战]()
