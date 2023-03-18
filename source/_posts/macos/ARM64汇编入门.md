---
title: ARM64汇编入门
date: 2023-03-18 15:02:42
tags:
- arm64
categories:
- 汇编
---

# `ARM` 指令概要介绍

- `A64` 指令集只能运行在 `aarch64` 环境中
- 所有的A64汇编指令都是 `32bits` 宽
- `A64` 支持全部是大写或者全部是小写的书写方式


寄存器名：


| Name  | Size    | Encoding | Description                  |
| :---  | :---    | :---     | :---                         |
| Wn    | 32 bits | 0-30     | Genral-purpose register 0-30 |
| Xn    | 64 bits | 0-30     | Genral-purpose register 0-30 |
| WZR   | 32 bits | 31       | Zero register                |
| XZR   | 64 bits | 31       | Zero register                |
| WPS   | 32 bits | 31       | Current stack pointer        |
| SP    | 64 bits | 31       | Current stack pointer        |

<!--more-->

## `ARM` 指令的分类

- 内存加载和存储指令
- 多字节内存加载和存储
- 算术和位移指令
- 移位操作
- 位操作
- 条件操作
- 跳转指令
- 独占访存指令
- 内存屏障指令
- 异常处理指令
- 系统寄存器访问指令


## `ARM` 指令的一般编码格式
一条典型的 `ARM64` 指令语法格式如下所示：

```
<opcode>{<cond>}{S} <Rd>, <Rn>, <shifter_operand>
```

- `<opcode>`：是指令助记符，如 `ADD` 表示算术加操作指令。
- `{<cond>}`：表示指令执行的条件。
- `{S}`：决定指令的操作是否影响 `CPSR` 的值。
- `<Rd>`：表示目标寄存器。
- `<Rn>`：表示包含第 `1` 个操作数的寄存器。
- `<shifter_operand>`：表示第 `2` 个操作数。
