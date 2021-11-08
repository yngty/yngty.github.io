---
title: Effective C++ 16：使用同样的形式来new和delete
date: 2021-11-08 09:23:53
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 16: Use the same form in corresponding uses of new and delete.

**如果你用 `new` 申请了动态内存，请用 `delete` 来销毁；如果你用 `new xx[]` 申请了动态内存，请用 `delete[]` 来销毁**: 

举个栗子：
```c++
std::string* stringPtrl = new std::string;
std::string* stringPtr2 = new std::string[lOO];
...
delete stringptrl;      // 删除一个对象
delete [] stringPtr2;  // 删除一个由对象组成的数组
```

上面很容易理解但需要注意`typedef`:

```c++
typedef std::string AddressLines[4];    //每个人的地址有四行，
                                        //每行是一个string
```

由于 `AddressLines` 是个数组，如果这样使用 `new`:

```c++
std::string *pal = new AddressLines;     //注意. "new AddressLines" 返回
                                         //一个 string*，就像 "new string[4]" 一样
```
那就必须匹配 "**数组形式**"的 `delete`:
```c++
delete pal;         //行为未有定义!
delete [] pal;     //很好。
```

为避免诸如此类的错误，最好尽量不要对数组形式做 `typedefs` 动作。可以使用更加面向对象的`vector`、`string`等对象。