---
title: Effective C++ 21：需要返回对象时，不要返回引用
date: 2022-03-23 10:43:36
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Don't to return a reference when you must return an object.

Item 20 中提到，多数情况下传引用比传值更好。但不要无脑追求这一点，一定不要返回空引用或指针。

举个栗子：

```c++
class Rational{
  int n, d;
public:
  Raitonal(int numerator=0, int denominator=1);
};

// 返回值为什么是const请参考Item 3
friend const Rational operator*(const Rational& lhs, const Rational& rhs);

Rational a, b;
Rational c = a*b;
```
这个版本的 `operator*` 返回的是一个实例，`a*b`时便会调用`operator*()`， 返回值被拷贝后用来初始化`c`。

不考虑编译器优化和 `C11` 的 `move` ,这个过程涉及到多个构造和析构过程：

1. `operator*`调用结束前，返回值被拷贝，调用拷贝构造函数
2. `operator*`调用结束后，返回值被析构
3. `c` 被初始化，调用拷贝构造函数

我们能否通过传递引用的方式来避免这些函数调用？这要求在函数中创建那个要被返回给调用者的对象，而函数只有两种办法来创建对象：在栈空间中创建、或者在堆中创建。

<!--more-->

在栈空间中创建显然是错误的：

```c++
const Rational& operator*(const Rational& lhs, const Rational& rhs){
  Rational result(lhs.n*rhs.n, lhs.d*rhs.d);
  return result;
}
```
我们的目标是要避免调用构造函数，而 `result` 却必须像任何对象一样地由构造函数构造起, 而且得到的 `result` 永远是空。因为 `result` 是一个 `local` 对象，当函数调用结束后 `result`即被销毁。

在堆中创建也会问题:

```c++
const Rational& operator*(const Rational& lhs, const Rational& rhs){
  Rational *result  = new Rational(lhs.n*rhs.n, lhs.d*rhs.d);
  return *result;
}
```

首先还是得必须付出一个"构造函数调用"代价， 并且谁去 `delete`?

```c++
Rational w, x, y, z;
w = x*y*z;
```

上面这样合理的代码都会导致内存泄露。

使用静态变量的方式：

```c++
const Rational& operator*(const Rational& lhs, const Rational& rhs){
    static Rational result; // static 对象，此函数将返回
    result = ... ; // lhs 乘以 rhs. 并将结果置于 result 内。
    return result;
}
```

静态变量首先便面临着线程安全问题，除此之外当我们需要不止一个的返回值同时存在时也会产生问题：

```c++
if((a*b) == (c*d)){
  // ...
}
```

如果operator*的返回值是静态变量，那么上述条件判断恒成立，因为等号两边是同一个对象。所以我们还是老老实实返回对象实例就好，并且考虑到编译器优化和`move`语意，拷贝构造返回值带来的代价没那么高。

**永远不要返回局部对象的引用或指针或堆空间的指针，如果需要多个返回对象时也不能是局部静态对象的指针或引用**。{% post_link effecitve-cpp/effective-cpp-4 Item:4 确定对象被使用前已先被初始化 %}， 对于单例模式，返回局部静态对象的引用也是合理的。

