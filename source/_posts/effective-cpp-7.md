---
title: Effective C++ 7：为多态基类声明 virtual 析构函数
date: 2021-10-25 13:09:45
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 7: Declare destructors virtual in polymorphic base classes.

析构函数声明为虚函数目的在于以基类指针调用析构函数时能够正确地析构子类部分的内存。 否则子类部分的内存将会泄漏，正确的用法如下：

```c++
// 基类
class TimeKeeper{
public:
    virtual ~TimeKeeper();
    ...
};
TimeKeeper *ptk = getTimeKeeper():  // 可能返回任何一种子类
...
delete ptk;
```

- polymorphic (带多态性质的) base classes 应该声明一个 virtual 析构函数。如果
class 带有任何 virtual 函数，它就应该拥有一个 virtual 析构函数。
- Classes 的设计目的如果不是作为 base classes 使用，或不是为了具备多态性
(polymorphically) ，就不该声明 virtual 析构函数。