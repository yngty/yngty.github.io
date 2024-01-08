---
title: C++ aggregate
date: 2024-01-08 13:55:00
categories:
- C/C++
---

# 什么是聚合类型(aggregate)

## 在 C++03 中的定义

- 不能有用户声明的构造函数
- 没有私有或受保护的非静态数据成员，可以拥有任意数量的私有和受保护的成员函数（但不能是构造函数）以及任意数量的私有或受保护的静态数据成员和静态成员函数
- 可以具有用户声明或用户定义的复制赋值运算符和或析构函数
- 没有基类
- 没有虚函数
- 数组是聚合，即使它是非聚合类类型的数组

<!--more-->
## 聚合的作用

### 可以使用 `{}` 初始化

#### 数组初始化

```c++
Type array_name[n] = {a1, a2, …, am};
```

- m == n
    - 数组的第 `i` 个 元素用 `ai` 初始化
- m < n 
    - 数组的前 `m` 个元素用 `a1`、 `a2`...`am` 初始化
    - 剩下的 `n - m` 个元素使用值初始化
        - 标量类型对象用 `0`初始化
        - 具有用户声明的默认构造函数的类类型的对象被值初始化时，将调用其默认构造函数
        - 隐式定义默认构造函数，则所有非静态成员都会递归地进行值初始化，(成员是引用的会初始化失败)
- m > n
    - 编译报错

```c++
class A
{
public:
  A(int) {} //no default constructor
};
class B
{
public:
  B() {} //default constructor available
};
int main()
{
  A a1[3] = {A(2), A(1), A(14)}; //OK n == m
  A a2[3] = {A(2)}; //ERROR A has no default constructor. Unable to value-initialize a2[1] and a2[2]
  B b1[3] = {B()}; //OK b1[1] and b1[2] are value initialized, in this case with the default-ctor
  int Array1[1000] = {0}; //All elements are initialized with 0;
  int Array2[1000] = {1}; //Attention: only the first element is 1, the rest are 0;
  bool Array3[1000] = {}; //the braces can be empty too. All elements initialized with false
  int Array4[1000]; //no initializer. This is different from an empty {} initializer in that
  //the elements in this case are not value-initialized, but have indeterminate values 
  //(unless, of course, Array4 is a global array)
  int array[2] = {1, 2, 3, 4}; //ERROR, too many initializers
}
```
### 类初始化

- 按照非静态数据成员在类定义中出现的顺序（根据定义它们都是公共的）来初始化非静态数据成员
- 如果初始化器的数量少于成员，则其余的将进行值初始化
- 如果无法对未显式初始化的成员之一进行值初始化(例如成员是引用)，则会出现编译时错误
- 如果初始化程序多于必要的数量，我们也会收到编译时错误

  ```c++
  struct X
  {
    int i1;
    int i2;
  };
  struct Y
  {
    char c;
    X x;
    int i[2];
    float f; 
  protected:
    static double d;
  private:
    void g(){}      
  }; 

  Y y = {'a', {10, 20}, {20, 30}};
  ```

## C++11 的变化

### 不能有用户提供的构造函数

以前聚合不能有用户声明的构造函数，但现在它不能有用户提供的构造函数。

```c++
struct Aggregate {
  Aggregate() = default; // 在 C++11 仍然是聚合类型
};
  ```
### 不能为非静态数据成员提供任何大括号或等号初始化程序

```c++
struct NotAggregate {
  int x = 5; // valid in C++11
  std::vector<int> s{1,2,3}; // also valid
};
  ```

## C++14 的变化

### 允许类内成员初始值

 [N3605: Member initializers and aggregates](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3605.html)

```c++
// 在C++11 不是聚合，在C++14 是聚合
struct Aggregate
{
  int a = 3;
  int b = 3;
};
  ```

## C++17 的变化

`C++17` 扩展并增强了聚合和聚合初始化。标准库现在还包含一个 `std::is_aggregate` 类型特征

### 聚合类现可以具有公共的非虚拟基类。

在新的扩展中，如果类存在继承关系，则额外满足以下条件：
- 必须是公开的基类，不能是私有或者受保护的基类
- 必须是非虚继承

此外，**不要求基类是聚合的**。如果它们不是聚合，则它们是列表初始化的。

聚合类的初始化顺序是按基类的声明顺序，然后按不是匿名联合成员的直接非静态数据成员声明顺序

```c++
struct B1 // 不是聚合类，有用户提供的构造函数
{
    int i1;
    B1(int a) : i1(a) { }
};
struct B2
{
    int i2;
    B2() = default;
};
struct M // 不是聚合类，有用户提供的构造函数
{
    int m;
    M(int a) : m(a) { }
};
struct C : B1, B2
{
    int j;
    M m;
    C() = default;
};
C c { { 1 }, { 2 }, 3, { 4 } };
cout
    << "is C aggregate?: " << (std::is_aggregate<C>::value ? 'Y' : 'N')
    << " i1: " << c.i1 << " i2: " << c.i2
    << " j: " << c.j << " m.m: " << c.m.m << endl;

//输出: is C aggregate?: Y, i1=1 i2=2 j=3 m.m=4
```

### 没有用户提供的、显式的或继承的构造函数

- 不允许显式构造函数

  ```c++
  struct D // not an aggregate
  {
      int i = 0;
      D() = default;
      explicit D(D const&) = default;
  };
  ```
- 不允许继承构造函数

  ```c++
  struct B1
  {
      int i1;
      B1() : i1(0) { }
  };
  struct C : B1 // not an aggregate
  {
      using B1::B1;
  };
  ```
### 扩展聚合类型的兼容问题

例如以下代码:
```c++
#include <iostream>
#include <string>

class BaseData {
  int data_;
public:
  int Get() { return data_; }
protected:
  BaseData() : data_(11) {}
};

class DerivedData : public BaseData {
public:
};

int main()
{
  DerivedData d{};
  std::cout << d.Get() << std::endl;
}
```
在 `c++17` 之前 `DerivedData` 不是聚合类型，所以会调用 `DerivedData` 的默认构造函数然后，调用 `BaseData` 的默认构造函数，虽然这里 `BaseData` 声明的是受保护的构造函数，但派生类是可以调用的，但在 `C++17` 之后发送了变化，`DerivedData` 是一个聚合类型，基类 `BaseData` 中的构造函数是受保护的关系，它**不允许**在聚合类型初始化中被调用导致编译失败，我可以可以通过添加一个默认构造函数使其不是聚合类型解决该问题。

## C++20 的变化

## 没有用户声明或继承的构造函数

又改回**没有用户声明**的构造函数了。因为没有用户定义的构造函数有时候会导致一些误会，例如下面的情况：

```c++
#include <iostream>
struct X {
  X() = default;
};

struct Y {
  Y() = delete;
};

int main() {
  std::cout << std::boolalpha 
      << "std::is_aggregate_v<X> : " << std::is_aggregate_v<X> << std::endl
      << "std::is_aggregate_v<Y> : " << std::is_aggregate_v<Y> << std::endl;
}

// 输出: std::is_aggregate_v<X> : true
//      std::is_aggregate_v<Y> : true
```

```c++
Y y1;   // 编译失败
Y y1{}; // 编译成功
```
除了删除默认构造函数，将其列入私有访问中也会有同样的问题，所以 `C++17` 增加了 `explicit` 修饰的构造函数，该类不是聚合类，但没有解决相同类型不同实例化方式表现不一致的尴尬问题。最后在 `C++20` 标准中禁止聚合类型使用用户声明的构造函数。


### 初始化
- 聚合类型对象的初始化可以用小括号列表来完成，其最终结果与大括号列表相同
- 另外带大括号的列表初始化是**不支持缩窄转换**的，但是带小括号的列表初始化却是支持缩窄转换的

```c++
struct X {
  int i;
  short f;
};

int main() {
    X x2{1, 7.0}; // 编译失败，7.0 从 double 转换到 short 是缩窄转换
    X x2(1, 7.0); // 编译成功， c++20 之前，编译失败
}
```
值得注意的是，这个规则的修改会改变一些旧代码的意义，比如我们经常用到的禁止复制构造的方法：

```c++
struct X {
  std::string s;
  std::vector<int> v;
  X() = default;
  X(const X&) = delete;
  X(X&&) = default;
};
```
在 `C++20 `中 `X` 不是聚合类，一个可行的解决方案是不要直接使用 `delete` 来删除复制构造函数，而是通过**加入或者继承**一个不可复制构造的类型来实现类型的不可复制。

```c++
struct X {
  std::string s;
  std::vector<int> v;
  [[no_unique_address]] NonCopyable nc;
};

// 或者

struct X : NonCopyable {
  std::string s;
  std::vector<int> v;
};
```

# 参考资料

* [What are aggregates and trivial types/PODs, and how/why are they special?](https://stackoverflow.com/questions/4178175/what-are-aggregates-and-trivial-types-pods-and-how-why-are-they-special)
* [现代 C++ 语言核心特性解析](https://book.douban.com/subject/35602582)