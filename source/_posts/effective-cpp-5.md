---
title: Effective C++ 5：了解c++默默编写并调用哪些函数
date: 2021-10-24 20:17:05
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 5: Know what functions C++ silently writes and calls

# 默认函数
在 `C++` 中，一个类有八个默认函数：

```c++
class Empty {
    Empty () {} //默认构造函数    
    Empty (const Empty &) {} // 默认拷贝构造函数
    Empty (const Empty &&) {} // 默认移动构造函数(`C++11`)
    ~Empty() {} // 默认析构函数
    Empty& operator=(const Empty&) {} // 默认重载赋值运算符函数
    Empty& operator=(const Empty&&){} // 默认重载移动赋值操作符函数函数
    Empty* operator &() {} // 默认重载取址运算符函数
    const Empty* operator &() const {} // 默认重载取址运算符 `const` 函数
};
```

# 调用时机

只有你需要用到这些函数并且你又没有显示的声明这些函数的时候，编译器才会贴心的自动声明相应的函数。

# 引用成员

如果你打算在一个“内含引用成员”或者“内含`const`成员”的类内支持赋值操作，就必须定义自己的默认拷贝赋值操作符。因为 `C++` 本身不允许引用改指不同的对象，也不允许更改 `const` 成员。

```c++
class Person {
public:
    string & name;
    Person(string &str):name(str) {}
};

string s1 = "hello", s2 = "world";
Person p1(s1), p2(s2);
p1 = p2;
```

```
error: object of type 'Person' cannot be assigned because its copy assignment operator is implicitly deleted
```