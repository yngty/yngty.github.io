---
title: Effective C++ 17：在单独的语句中将 new 的对象放入智能指针
date: 2021-11-08 09:53:13
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 17: Store newed objects in smart pointers in standalone statements.

**以单独的语句将 `new` 的对象放入智能指针内。这是为了由于其他表达式抛出异常而导致的资源泄漏**。

举个栗子：

```c++
processWidget(shared_ptr<Widget>(new Widget), priority());
```

上述代码中，在 `processWidget` 函数被调用之前参数会首先得到计算。可以认为包括三部分的过程：

1. 执行 `new Widget`
2. 构造 `shared_ptr<Widget>`
3. 调用 `priority()`

**因为C++不同于其他语言，函数参数的计算顺序很大程度上决定于编译器**编译器认为顺序应当是1, 3, 2，即：

1. 执行 `new Widget`
2. 调用 `priority()`
3. 构造 `shared_ptr<Widget>`

那么如果 `priority`抛出了异常，新的 `Widget` 便永远地找不回来了。虽然我们使用了智能指针，但资源还是泄漏了！

于是更加健壮的实现中，应当将创建资源和初始化智能指针的语句独立出来：

```c++
shared_ptr<Widget> pw = shared_ptr<Widget>(new Widget);
processWidget(pw, priority());
```