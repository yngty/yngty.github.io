---
title: 'std::make_shared vs. new'
date: 2023-02-07 10:30:24
tags:
- C++
- 智能指针
categories:
- C/C++
---


# 内存分配

`std::make_shared` 执行**一次**堆分配，而调用`std::shared_ptr` 构造函数执行**两次**。

在一个典型的实现中 `std::shared_ptr` 管理两个实体：

- 控制块（存储元数据，如引用计数、类型擦除删除器等）
- 被管理的对象

控制块是一个动态分配的对象，它包含：
- 指向托管对象的指针或托管对象本身；
- 删除器 (类型擦除)
- 分配器 (类型擦除)
- 拥有被管理对象的 `shared_ptr`的数量
- 引用托管对象的 `weak_ptr` 的数量 

`std::make_shared`执行一次堆分配，计算控制块和数据所需的总空间。在另一种情况 `std::shared_ptr<Obj>(new Obj("foo"))`下执行两次, `new Obj("foo")`为托管数据调用堆分配，`std::shared_ptr`构造函数为控制块执行另一个堆分配。

<!--more-->

# 异常安全

**自 `C++17` 以来，这不是问题，因为函数参数的评估顺序发生了变化。具体来说，函数的每个参数都需要在评估其他参数之前完全执行**。

考虑这个例子:

```
f(std::shared_ptr<int>(new int(42)), g())
```
因为 C++ 允许对子表达式进行任意顺序的计算，所以一种可能的顺序是：
1. `new int(42)`
2. `g()`
3. `std::shared_ptr<int>`

现在，假设我们在第 `2` 步抛出一个异常。然后我们丢失了在步骤 `1` 分配的内存，因为没有将原始指针传给 `std::shared_ptr`, 后面没有任何东西有机会清理它。

解决这个问题的一种方法是在不同的行上执行它们，这样就不会发生这种任意排序。

```
auto ptr = std::shared_ptr<int>(new int(42));
f(ptr, g());
```

更方便的的是采用 `std::make_shared`.

```
f(std::make_shared<int>(42), g())
```

# `std::make_shared` 的一些缺点

## `weak_ptr`内存保活

-  通过 `std::make_shared` 构造的智能指针, 当没有 `shared_ptr` 引用计数为 `0` 时只调用析构函数，`weak_ptr`引用计数为 `0` 时才释放内存块。


```c++
bool logging = false;
void* operator new(std::size_t size) {
    auto ptr = std::malloc(size);
    if (logging) {
        std::cout << "Allocated: " << (uintptr_t)ptr << std::endl;
    }
    return ptr;
}

void operator delete(void *ptr) noexcept  {
    std::free(ptr);
    if (logging) {
        std::cout << "Deallocated: " << (uintptr_t)ptr << std::endl;
    }
}
struct Widget {
    ~Widget() {
        std::cout << "Widget::~Widget()" << std::endl;
    }
    int data;
};
int main(int argc, char*argv[]) {
    logging = true;
    test(true);
    std::cout << "---------------------\n";
    test(false);
    return 0;
}
```

```
Allocated: 105553162522944 //分配一次
Widget::~Widget() // 没有 shared_ptr 指针，只调用析构函数
No std::shared_ptr's anymore.
Deallocated: 105553162522944 //没有 `weak_ptr` 释放整个内存块
No std::weak_ptr's anymore.
---------------------
Allocated: 105553164599312
Allocated: 105553162522944 //分配两次
Widget::~Widget()
Deallocated: 105553164599312 //立即释放
No std::shared_ptr's anymore.
Deallocated: 105553162522944
No std::weak_ptr's anymore.
```

##  无法访问公共构造函数

```c++
class A
{
public:
    A(): val(0){}

    // make_shared 无法调用 A(int) 
    // std::shared_ptr<A> createNext(){ 
    //     return std::make_shared<A>(val+1); 
    //}
    
    // 可以调用 A(int) 
    std::shared_ptr<A> createNext(){ 
        return std::shared_ptr<A>(new A(val+1)); 
    }
private:
    int val;
    A(int v): val(v){}
};
```

# 参考资料

* [Difference in make_shared and normal shared_ptr in C++](https://stackoverflow.com/questions/20895648/difference-in-make-shared-and-normal-shared-ptr-in-c)
* [std::make_shared 与 new](https://www.gamedev.net/forums/topic/695796-stdmake_shared-vs-new/)
* [make_shared#Notes](https://zh.cppreference.com/w/cpp/memory/shared_ptr/make_shared#Notes)