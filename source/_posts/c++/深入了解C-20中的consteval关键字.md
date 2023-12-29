---
title: 深入了解 C++20 中的 consteval
date: 2023-12-29 18:16:18
tags:
- C++20
- const
categories:
- C/C++
---

# 简介

用最简单的术语来说，一个只能应用于函数的 `consteval` 关键字, 保证它产生一个编译时间常数。否则会导致编译错误。

`cppreference` 页面对 `consteval` 说明符有如下描述：
>`consteval` 指定函数是立即函数，也就是说，对该函数的每次调用都必须产生一个编译时常量

什么是立即函数？

- 不能是协程
- 函数主体中不能有 `throw` 语句
- 不能有 `goto` 语句或标签语句，除了 `case` 和 `default`
- 参数和返回类型必须是[LiteralType](https://en.cppreference.com/w/cpp/named_req/LiteralType),简单地说，是一个可以在编译时计算的类型（比如所有可以在 `constexpr` 上下文中使用的类型）

<!--more-->

# `consteval` 和 `constexpr` 方法的区别

他们最大的区别：`consteval` 保证编译时生成，不能编译时生成时会报错，但 `constexpr` 不一定，当编译时不能生成就转为运行时函数。

# 从汇编看 `consteval` 和 `constexpr` 函数

## 定义函数

对于定义的普通函数、 `consteval` 函数、 `constexpr` 函数, 只有普通函数会生成汇编代码
![consteval1](/images/consteval1.png)
## 在编译时上下文调用 `consteval` 和 `constexpr` 函数

在编译时直接生成结果，不会生成对应的函数汇编代码和调用汇编指令
![consteval2](/images/consteval2.png)

## 在非编译时上下文调用 `consteval` 和 `constexpr` 函数

会生成 `constexpr`函数的汇编代码和调用函数汇编指令
![consteval3](/images/consteval3.png)


# 总结

- `consteval` 说明符只能应用于函数和构造函数
- 带有 `consteval` 的函数称为立即函数
- `constexpr` 函数在编译时上下文中执行时，与`consteval` 函数行为相同
- 当 `consteval` 函数不能产生编译时错误时，会导致错误，而对于 `constexpr` 函数则不会。
- 当需要对函数的编译时求值进行保证时，首选 `consteval`
- 优先使用 `consteval` 函数而不是预处理器宏函数。 