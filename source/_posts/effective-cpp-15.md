---
title: Effective C++ 15：在资源管理类中提供对原始资源的访问
date: 2021-11-05 15:58:35
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 15: Provide access to raw resources in resource-managing classes.

`APIs` 往往要求访问原始资源(`raw resources`)，所以每一个RAII class 应该**提供提供对原始资源访问的方法。获取资源的方式有两类：隐式地获取和显式地获取。** 显式的资源获取会更安全，它最小化了无意中进行类型转换的机会。

- **显示获取**

`shared_ptr` 提供了 `get` 方法来得到资源。

```c++
shared_ptr<Investment> pInv;
void daysHeld(Investment *pi);

int days = daysHeld(pInv.get());
```

为了让 `pInv` 表现地更像一个指针，`shared_ptr`还重载了解引用运算符（`dereferencing operator`） `operator->`和 `operator*`：

```c++
class Investment{
public: 
    bool isTaxFree() const;
};
shared_ptr<Investment> pi1(createInvestment());

bool taxable1 = !(pi1->isTaxFree());
bool texable2 = !((*pi1).isTaxFree());
```

我们封装了Font来管理资源：

```c++
class Font{
FontHandle f;
public:
    explicit Font(FontHandle fh): f(fh){}
    ~Font(){ releaseFont(f); };
    FontHandle get() const { return f; }
};
```
通过get方法来访问FontHandle：

```c++
Font f(getFont());
int newFontSize;
changeFontSize(f.get(), newFontSize);
```

- **隐式地获取**

可以隐式类型转换运算符将 `Font` 转换为 `FontHandle`:

```c++
class Font{
    operator FontHandle() const{ return f;}
};

changeFontSize(f, newFontSize);
```

然而问题也随之出现：

```c++
FontHandle h2 = f1;
```
无意间 `h2` 并没有被资源管理起来，这将会引发意外的资源泄漏。所以隐式转换在提供便利的同时， 也引起了资源泄漏的风险。 

