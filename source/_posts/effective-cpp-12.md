---
title: Effective C++ 12：复制对象时勿忘其每一个成分
date: 2021-11-02 17:36:33
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 12: Copy all parts of an object

正确拷贝函数实现：

```c++
class Customer{
  string name;
public:
  Customer(const Customer& rhs): name(rhs.name){}
  Customer& operator=(const Customer& rhs){
    name = rhs.name;                     // copy rhs's data
    return *this;                        // see Item 10
  }  
};
```

### 情形一： 新添加了一个数据成员，忘记了更新拷贝函数

```c++
class Customer{
  string name;
  Date lastTransaction;
public:
  Customer(const Customer& rhs): name(rhs.name){}
  Customer& operator=(const Customer& rhs){
    name = rhs.name;                     // copy rhs's data
    return *this;                        // see Item 10
  }  
};
```

这时 `lastTransaction` 便被你忽略了，编译器也不会给出任何警告（即使在最高警告级别）

### 情形二： 继承父类忘记了拷贝父类的部分

```c++
class PriorityCustomer: public Customer {
    int priority;
public:
  PriorityCustomer(const PriorityCustomer& rhs)
  : priority(rhs.priority){}
  
  PriorityCustomer& 
  operator=(const PriorityCustomer& rhs){
    priority = rhs.priority;
  }  
};
```

正确写法:

```c++
class PriorityCustomer: public Customer {
    int priority;
public:
  PriorityCustomer(const PriorityCustomer& rhs)
  : Customer(rhs), priority(rhs.priority){}
  
  PriorityCustomer& 
  operator=(const PriorityCustomer& rhs){
    Customer::operator=(rhs);
    priority = rhs.priority;
  }  
};
```