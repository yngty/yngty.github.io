---
title: 网络工）ttelnet、 nc、 netstat、ss
date: 2024-03-20 09:42:47
tags:
- tools
- network
categories:
- Network
---

# `telnet`

## 检查端口是否打开

```shell
telnet [domainname or ip] [port]

telnet 220.181.57.216 80
```

# `netcat`

## 当服务器

```shell
nc -l [port]

nc -l 9090
```
## 连接服务器

```shell
nc [host or ip] [port]

nc 10.211.55.5 9090
```

## 查看远程端口是否打开

```shell
nc -zv [host or ip] [port]
```

其中 `-z` 参数表示不发送任何数据包，`tcp` 三次握手完后自动退出进程。有了 `-v` 参数则会输出更多详细信息（`verbose`）


# netstat

`netstat` 用来显示套接字的状态。


## 列出所有套接字

```shell
netstat -a
```
- `-a` 命令可以输出所有的套接字，包括监听的和未监听的套接字。 示例输出：

```shell
➜  ~ netstat -a
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4       0      0  yanghao.51628          43.134.115.68.https    ESTABLISHED
tcp4       0      0  localhost.hydap        localhost.51627        ESTABLISHED
tcp4       0      0  localhost.51627        localhost.hydap        ESTABLISHED
tcp4       0      0  localhost.ddi-tcp-2    localhost.51626        ESTABLISHED
tcp4       0      0  localhost.51626        localhost.ddi-tcp-2    ESTABLISHED
```

## 只列出 `TCP` 连接
```shell
netstat -at
```

## 只列出 `UDP` 连接
```shell
netstat -au
```
## 只列出处于监听状态的连接

```shell
netstat -l
```
- `-l` 选项用来指定处于 `LISTEN` 状态的连接
## 禁用端口 和 IP 映射

```shell
netstat -ltn
```
- 常用端口都被映射为了名字，比如 `22` 端口输出显示为 `ssh`，`8080` 端口被映射为 `webcache`。大部分情况下，我们并不想 `netstat` 帮我们做这样的事情，可以加上 `-n` 禁用
## 显示进程

```shell
netstat -ltnp
```
- `-p` 命令可以显示连接归属的进程信息，在查看端口被哪个进程占用时非常有用.

## 显示所有的网卡信息

```shell
netstat -i
```

## `ss`
