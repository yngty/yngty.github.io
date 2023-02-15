---
title: Effective C++ 14：在资源管理类中小心 copying 行为
date: 2021-11-05 00:00:01
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 14: Think carefully about copying behavior in resource-managing classes.

设计一个 **`RAII`** 对象：

```c++
class Lock {
public:
    explicit Lock(Mutex *pm):mutexPtr(pm){
        lock(mutexPtr);
    }
    ~Lock(){ unlock(mutexPtr); }
private:
    Mutex *mutexPtr;
};
```
客户对`Lock`的使用：

```c++
Mutex m;
...
{
    Lock ml(&m);    
    ...
}
```

当一个 **`RAII`** 对象被复制，会发生什么事？ 不确定？

```c++
Lock ml1(&m);
Lock ml2(&ml1)
```

记住**资源管理对象的拷贝行为取决于资源本身的拷贝行为，同时资源管理对象也可以根据业务需要来决定自己的拷贝行为**。一般有如下四种方式：

- **禁止复制**。参考{% post_link effecitve-cpp/effective-cpp-6 若不想使用编译器自动生成的函数，就该明确拒绝 %}。对Lock而言看起来是这样：

    ```c++
    class Lock : private Uncopyable {
    public:
        ...
    }
    ```
- **引用计数**，采用 `shared_ptr` 的逻辑。`shared_ptr` 构造函数提供了第二个参数 `deleter`，当引用计数到 `0` 时被调用。 所以 `Lock` 可以通过聚合一个 `shared_ptr` 成员来实现引用计数：
    ```c++
    class Lock{
    public: 
        explicit Lock(Mutex *pm): mutexPtr(pm, unlock){
            lock(mutexPtr.get());
        }
    private: 
        std::shared_ptr<Mutex> mutexPtr; //shared_ptr替换 raw pointer
    };
    ```
     `Lock` 的析构会引起 `mutexPtr` 的析构，而 `mutexPtr` 计数到0时`unlock(mutexPtr.get())` 会被调用。

- **拷贝底部资源**。复制资源管理对象时，进行的是**深拷贝**。比如 `string` 的行为：内存存有指向对空间的指针，当它被复制时会复制那片空间。
- **转移底部资源的拥有权**。`auto_ptr` 就是这样做的，把资源移交给另一个资源管理对象，自己的资源置空。