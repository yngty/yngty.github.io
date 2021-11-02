---
title: Effective C++ 11：赋值运算符需要考虑自我赋值问题
date: 2021-11-01 13:12:55
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 11: Handle assignment to self in operator=

我们在重载一个类的赋值运算符时要考虑自我赋值的问题。有了指针和引用自我赋值不总是第一时间能够识别出来。

```c++

a[i] = a[j];

*px = *py;

class Base { ... };
class Derived: public Base { ... };
void doSomething(const Base& rb, Derived* pd);// rb和女pd 有可能其实是同一对象
rb = pd;
```

自我赋值主要考虑到 **自我赋值安全性** 和 **异常安全性**

```c++
class Bitmap { ... };
class Widget {
private:
    Bitmap* pb; //指针，指向一个从heap 分配而得的对象
};
```

既不自我赋值安全性也不异常安全性, 当 rhs == *this时，delete pb使得rhs.pb成为空值，接下来 new 的数据便是空的。

```c++
Widget& Widget::operator=(const Widget& rhs) {
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```

判断两个地址是否相同，如果是自我赋值，就不做任何事。但开始就delete pb， 但 new 出现异常， pb就会置空出现风险。  

```c++
Widget& Widget::operator=(const Widget& rhs) {
    if (this == &rhs) return this;  // 证同测试
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```

在C++中**仔细地排列语句顺序**通常可以达到异常安全， 比如我们可以先申请空间，最后再delete：

```c++
Widget& Widget::operator=(const Widget& rhs) {
    Bitmap *pOrig = pb;  
    pb = new Bitmap(*rhs.pb);
    delete pOrig;
    return *this;
}
```

一个更加通用的技术便是复制和交换（copy and swap）：

```c++
class Widget {
    void swap(Widget& rhs); // 交换*this rhs 的数据
};

Widget& Widget::operator=(const Widget& rhs)
{
    Widget temp(rhs); //rhs 数据制作一份复件(副本)
    swap (temp); //*this 数据和上述复件的数据交换
    return *this;
}
```