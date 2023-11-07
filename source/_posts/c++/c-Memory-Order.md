---
title: c++ Memory Order
date: 2023-11-07 16:20:11
tags:
- Memory Order
categories:
- C/C++
---

# 背景

高级语言经过编译器将源码转为机器指令运行，其中的运行顺序和代码中的顺序有很大差异，主要是下面三个原因: 
- 编译器重排
- `CPU` 乱序执行
- 存储器硬件设计，不同线程看到的顺序不一致。

在 `c++` 中线程同步只有两种方式：
- 原子变量进行同步
- 锁(`Mutex`)

这里我们主要讨论原子变量的操作。

# Memory Order
`C++11` 规定了六种不同的 `memory order`:

- `Relaxed`
- `Consume`
- `Acquire`
- `Release`
- `Acquire-Release`
- `Sequential Consistent`
<!--more-->
对应着 `std::memory_order` 枚举值:
- `std::memory_order_relaxed`
- `std::memory_order_consume`
- `std::memory_order_acquire`
- `std::memory_order_release`
- `std::memory_order_acq_rel`
- `std::memory_order_seq_cst`

对一个原子变量操作时可以传入一个  `std::memory_order` 枚举，指明这个原子操作需要满足的 `memory order`。没有传入默认为 `std::memory_order_seq_cst`。

## `Relaxed`

最弱内存序，单纯的原子操作，没有线程间同步节点的作用。即：

- 在一个 `relaxed` 写操作之前的写操作, 将不保证能被其他也对同一个原子变量的 `relaxed` 读操作看到。
- 在一个 `relaxed` 读操作之后的读操作, 也不保证能看到被其他也对同一个原子变量的 `relaxed` 写操作之前的写操作。


## `Consume`

`Consume` 仅对原子**读**操作有效。

我们首先理解什么是操作数之间的**数据依赖**: 

对于操作 `A` 和操作 `B`，如果操作 `A` 先于操作 `B` 发生，则有三种情况会使得操作 `B` 数据依赖于 操作 `A`：

- `A` 的值被用作 `B` 的运算数，**除了**
  - `B` 是对 std::kill_dependency 的调用
  - `A` 是内建 `&&`、`||`、`?:` 或 `,` 运算符的左运算数。
- `A` 写入标量对象 `M`，`B` 从 `M` 读取
- 存在第三个操作 `X` 数据依赖于操作 `A`，操作 `B` 又数据依赖于操作 `X`

`acquire` 要求线程 `B` 能够看到线程 `A` 中在 `release` 操作之前的**具有数据依赖关系的写操作**



## `Acquire`

`Acquire` 仅对原子**读**操作有效。`acquire` 与 `consume` 唯一的区别是 `acquire` 要求线程 `B` 能够看到线程 `A` 中在 `release` 操作之前的**所有写操作**，而不仅仅是与写原子变量具有数据依赖关系的写操作。



## `Release`
`Release` 仅对原子**写**操作有效。`Release` 操作通常是与 `consume` 或 `acquire` 操作配对的。


## `Acquire-Release`

- 对于一个原子读操作，该操作都是 `acquire` 操作
- 对于一个原子写操作，该操作是 `release` 操作
- 对于一个既有读又有写的原子操作，该操作既是 `acquire` 操作也是 `release` 操作, 例如 `compare-and-swap `操作、read-modify-write` 操作

## `Sequential Consistent`

- 对于一个原子读操作，该操作都是 `acquire` 操作
- 对于一个原子写操作，该操作是 `release` 操作
- 对于一个既有读又有写的原子操作，该操作既是 `acquire` 操作也是 `release` 操作
- 程序内所有线程在使用 `sequential consistent` 操作原子变量时，必须以**一致的顺序**看到程序内的所有 `sequential consistent` 操作


# 实现方式

在 `x86_64`平台主流实现方式：

- 限制线程同步节点前后的代码重排
  - Consume
    - 所有的在 `release` 操作之前的、与 `release` 操作具有数据依赖关系的写操作不能被移动到 `release` 操作之后
    - 所有的在 `consume` 操作之后的、与 `consume` 操作具有数据依赖关系的读操作不能被移动到 `consume` 操作之后
  - Acquire
    - 所有的在 `release` 操作附近之前的写操作不能被移动到 `release` 操作之后
    - 所有的在 `acquire` 操作附近之后的读操作不能被移动到 `acquire` 操作之前
  - Release
    - 所有在 `release` 操作附近之前的写操作均不能被移动到 `release` 操作之后
  - Acquire-Release
    - 所有的在 `acquire-release` 操作附近之前的写操作不能被移动到 `acquire-release` 操作之后
    - 所有的在` acquire-release` 操作附近之后的读操作不能被移动到 `acquire-release` 操作之前
- 利用硬件特性，生成带 `lock` 前缀的操作指令
  - Sequential Consistent