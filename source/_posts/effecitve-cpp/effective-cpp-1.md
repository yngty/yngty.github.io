---
title: Effective C++ 1：将C++视作一系列的语言
date: 2020-12-07 20:40:56
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 1: View C++ as a federation of languages

一开始，`Ｃ++` 只是 `Ｃ` 加上一些面向对象特性，`Ｃ++` 最初的名称 `Ｃ with Classes` 也反映了这个血缘关系。现在这个语言逐渐成熟，已经是一个**多重泛型编程语言**(`multiparadigm programming language`)。同时支持过程形式(`procedural`)、面向对象形式(`object-oriented`)、函数形式(`functional`)、泛型形式(`generic`)、元编程形式(`metaprogramming`)

将 `C++` 视为一个由相关语言组成的联邦而非单一的语言。

`C++` 主要４个子语言：

- `C`。说到底Ｃ++仍是以Ｃ为基础。许多时候Ｃ++对问题的解法其实不过就是较高级的Ｃ的解法如`item2`、`item13`。当只使用`C++`中`C`的那部分语法，　会发现`C`语言的缺陷：没有模板、没有异常、没有重载。
- `Object-Oriented`。面向对象程序设计也是`C++`的设计初衷：构造与析构、封装与继承、多态、动态绑定的虚函数。
- `Template C++`。这是C++的泛型编程部分，大多数程序员经验最少的部分。**TMP模板元编程**（`template metaprogramming`）也是一个新兴的程序设计范式。
- `STL`。`STL`是一个特殊的模板库，它将容器、迭代器和算法优雅地结合在一起。

`C++` 程序设计的惯例并非一成不变，而是取决于你使用 `C++` 语言的哪一部分。例如， 在基于C语言的程序设计中，基本类型传参时传值比传引用更有效率。 然而当你接触 `Object-Oriented C++` 时会发现，传常量指针是更好的选择。运用`Template C++`时尤其如此，因为彼时你甚至不知道所处理的对象的类型。 但是你如果又碰到了`STL`，其中的迭代器和函数对象都是基于`C`语言的指针而设计的， 这时又回到了原来的规则：传值比传引用更好。
