---
title: Effective C++ 4：确定对象被使用前已先被初始化
date: 2021-01-13 22:41:25
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Make sure that objects are initialized before they're used.

# 手工初始化内置对象

为内置对象进行手工初始化，因为`C++`不保证初始化他们。

```c++
int x = 0;                                  //对 int 进行手工初始化
const char *text = "A C-style string";      //对指针进行手工初始化

double d;
std::cin >> d;                              //以读取 input stream 的方式完成初始化
```

# 构造函数最好使用成员初值列

```c++
class PhoneNumber { ... }

class ABEntry {
public:
    ABEntry(const std::string &name, const std::string &address, const std::list<PhoneNumber> &phones);
private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int numTimesConsulted;
}

ABEntry::ABEntry(const std::string &name, const std::string &address, const std::list<PhoneNumber> &phones) {
    theName = name;             //这些都是赋值
    theAddress = address;       //而非初始化
    thePhones = phones;
    numTimesConsulted = 0; 
}
```

构造函数最好使用成员初值列，而不要在构造函数本体内使用赋值操作。初值列列出的成员变量，其排列次序应该和他们在`class`中的声明次序相同。

```c++
ABEntry::ABEntry(const std::string &name, const std::string &address, const std::list<PhoneNumber> &phones) : theName(name), theAddress(address), thePhones(phones), numTimesConsulted(0) {}
```

# `local static` 对象替换 `non-local static` 对象。

为免除”跨单元之初始化次序“问题，请以 `local static` 对象替换 `non-local static` 对象。

```c++ 
class FileSystem {
public:
    ...
    std::size_t numDisks() const;
    ...
}
extern FileSystem tfs;     
```

```c++
class Directory {
public:
    Directory( params );
    ...
}

Directory::Directory( params) 
{
    ...
    std::size_t disks = tfs.numDisks();
    ...    
}
```
客户使用使用：
```c++
Directory tempDir( params );
```
现在初始化次序的重要性体现出来了，除非 `tfs` 在 `tempDir` 之前先被初始化，否则`tempDir`的构造函数会用到尚未初始化的`tfs`。但`tfs`和`tempDir`是不同的人在不同的时间于不同的源文件建立起来的，它们是定义于不同编译单元内的 `non-local static` 对象。它们初始化相对次序并无明确定义。但我们可以将 `local static` 对象替换`non-local static` 对象来解决。这也是**Singleton**模式的常见实现手法。

这个手法的基础在于：C++保证，函数内的 `local static` 对象会在调用该函数时首次遇上该对象的定义式时被初始化。

```c++ 
class FileSystem { ... }

FileSystem& tfs() 
{
    static FileSystem fs;
    return fs;
}    
```

```c++
class Directory { ... }

Directory::Directory( params) 
{
    ...
    std::size_t disks = tfs().numDisks();
    ...    
}

Directory& tempDir()
{
    static Directory td;
    return td;
}
```

