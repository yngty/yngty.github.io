---
title: Effective C++ 6：若不想使用编译器自动生成的函数，就该明确拒绝
date: 2021-10-25 11:32:26
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 6: Explicitly disallow the use of compiler-generated functions you do not want.

在C++中，编译器会自动生成一些你没有显式定义的函数。可以参考:{% post_link effective-cpp-5 了解c++默默编写并调用哪些函数 %}
然而有时候我们希望禁用掉这些函数，可以通过把自动生成的函数设为 `private` 来禁用它或者在 `c++11` 中使用 `delete` 关键字。

比如我们禁用拷贝的功能：

```c++
class HomeForSale
{
public:
    ...
    
private:
    HomeForSale(const HomeForSale &);  // 只有声明
    HomeForSale& operator=(const HomeForSale&) = delete； // c++11

};
```

我们可以专门设计一个阻止`copying` 的类

```c++
namespace noncopyable_ {
    class noncopyable {
        protected:
            noncopyable() {}
            ~noncopyable(){}
            /** C++11
            noncopyable() = default;
            ~noncopyable() = default;
            */
        private:
            noncopyable(const noncopyable&);
            noncopyable& operator=( const noncopyable& );
            /** C++11
            noncopyable( const noncopyable& ) = delete;
            noncopyable& operator=( const noncopyable& ) = delete;
            */
    };
}
```

```c++
class HomeForSale : private noncopyable_::noncopyable
{
};

HomeForSale p1, p2;
p1 = p2;

error: object of type 'HomeForSale' cannot be assigned because its copy assignment operator is implicitly deleted
    p1 = p2;
       ^
```