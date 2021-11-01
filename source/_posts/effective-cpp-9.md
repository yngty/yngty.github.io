---
title: Effective C++ 9：绝不在构造和析构过程中调用 virtual 函数
date: 2021-11-01 12:03:51
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 9: Never call virtual functions during construction or destruction.

在构造和析构期间不要调用 `virtual` 函数，因为这类调用不会下降至 `derived class`
(比起当前执行构造函数和析构函数的那层)。

```c++
class Transaction {                               // base class for all
public:                                           // transactions
    Transaction(){                                // base class ctor           
        logTransaction();                         // as final action, log this               
    }
    virtual void logTransaction() const = 0;      // make type-dependent
};
  
class BuyTransaction: public Transaction {        // derived class
public:
    virtual void logTransaction() const;          // how to log trans-
};
...
BuyTransaction b;
```

`b` 在构造时，调用到父类Transaction的构造函数，其中对 `logTransaction` 的调用会被解析到 `Transaction` 类。 那是一个纯虚函数，因此程序会非正常退出。

在`derived class` 对象的 `base class` 构造期间，对象的类型是 `base class` 而不是 `derived classo` 不只 `virtual` 函数会被编译器解析至(resolve to) `base class` ，若使用运行期类型信息 `RTTI`(runtime type information, 例如 `dynamic_cast`  `typeid`) ，也会把对象视为 `base class` 类型。

```c++
class Transaction{
public:
    Transaction(){
        cout<<typeid(this).name()<<endl;
    }
};
class BuyTransaction: public Transaction{
public:
    BuyTransaction(){
        cout<<typeid(this).name()<<endl;
    }
};
void main(){
    BuyTransaction b;
}
```

输出

```
P11Transaction
P14BuyTransaction
```

**相同道理也适用于析构函数.**