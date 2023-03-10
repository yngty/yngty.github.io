---
title: Effective C++ 24：若所有参数皆需类型转换，请采用非成员函数
date: 2022-04-09 22:39:25
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 24: Declare non-member functions when type conversions should apply all parameters.

令 `classes` 支持隐式转换通常是糟糕的设计，但也有例外，最常见的是在建立数值类型时。 比如设计一个有理数 `class` 允许整数隐式转换。

<!--more-->

```c++
class Rational {
public:
    Rational(int numerator = 0, int denominator = 1); //构造函数刻意不使用 explicit; 允许 int-to-Rational 隐式转换。

    int numerator() const;
    int denominator() const;
private:
    ...
};
```

这时我们想设计一个乘法，该使用 `member` 函数，还是 `non-member` 函数， 还是 `non-member-friend` 函数？

我们先采用 `member` 函数看有什么问题？

```c++
class Rational {
public:
    ...

    const Rational operator* (const Rational& rhs) const;
};
```

我们使用如下没有什么问题：

```c++
Rational oneEighth(1, 8);
Rational oneHalf(1, 2);

Rational result = oneEighth * oneHalf; //ok
result = result * oneEighth;  // ok

```

但当我们想支持混合运算，那 `Rational` 和 `ints` 相乘, 就只有一半行的通。


```c++
result = oneHalf * 2; //ok  隐式转换
result = 2 * oneHalf;  // no
```

当我们设计成 `non-member` 函数就都支持：
```c++
class Rational {
    ...
};


const Rational operator* (const Rational& lhs, const Rational& rhs) {
    return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator(), rhs.denominator());
}
    
```

```c++
Rational oneFourth(1, 4);
Rational result;
result = oneFourth * 2;  // ok
result = 2 * oneFourth;  // ok 
```

**如果需要为某个函数的所有参数进行类型转换，那这个函数必须是 `non-member`**