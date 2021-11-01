---
title: Effective C++ 10：赋值运算符要返回自己的引用 
date: 2021-11-01 12:59:28
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 10：Have assignment operators return a reference to *this.

赋值运算符要返回自己的引用只是个协议，并无强制性。这份协议被所有内置类型和标准程序库提供的类型如`string`, `vector`, `complex` `std::shared_ptr`等共同遵守。可以用来支持链式的赋值语句。

```c++
int x, y, z;
x = y = z = 15; //赋值连锁形式
```

相当于:

```c++
x = ( y = ( z = 15 ) );
```

我们自定义的对象最好也能支持链式的赋值，这需要重载=运算符时返回当前对象的引用：

```c++
class Widget {
public:
    Widget& operator=(const Widget& rhs){   
      return *this;                         
    }

    //这个协议不仅适用于以上的标准赋值形式，也适用于所有赋值相关运算 +=, -=, *=, etc.
    Widget& operator+=(const Widget& rhs){  
       return *this;
    }
};
```
