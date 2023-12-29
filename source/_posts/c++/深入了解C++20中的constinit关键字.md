---
title: 深入了解 C++20 中的 constinit
date: 2023-12-29 17:41:16
tags:
- C++20
- const
categories:
- C/C++
---

在 `C++` 中，存储变量的方式有几种：

- 存储期（`Storage Duration`）：
存储期是指变量在程序中存在的时间段。在 `C++` 中，有三种主要的存储期：

    - 自动存储期（`Automatic Storage Duration`）：变量在函数或代码块执行时创建，函数执行结束时销毁。
    - 动态存储期（`Dynamic Storage Duration`）： 使用 `new` 或 `malloc` 分配的内存，直到使用 `delete` 或 `free` 手动释放为止。
    - 静态存储期（`Static Storage Duration`）： 变量在程序启动时创建，在整个程序运行期间都存在，直到程序结束才销毁。

- 静态存储变量：

静态存储变量是在程序启动时创建，一直存在于整个程序运行期间的变量。这类变量有两种主要形式：
    - 全局变量（`Global Variables`）： 在函数外部声明的变量，可以被程序中的所有函数访问。
    - 静态局部变量（`Static Local Variables`）： 在函数内部使用 `static` 关键字声明的变量，与自动存储期变量不同，它在函数调用之间保持其值。

在 `C++` 中，我们经常使用静态存储期变量，包括全局变量和使用 `static` 关键字声明的局部变量。然而，这些变量并不保证在程序执行前被初始化，除非它们被声明为 `const` 常量。为了解决这一问题，`C++20` 引入了 `constinit` 关键字，它为我们提供了一种保证变量在程序启动时被初始化的方式，从而增强了可预测性和可靠性。

尽管 `constinit` 确保变量在程序启动时被初始化，但这并不意味着这些变量是不可修改的常量。相反，这个关键字允许变量在初始化后在运行时或编译时上下文中被修改。

因此，`constinit` 关键字为我们提供了一种在使用静态存储期变量时获得初始化保证的方法，同时允许在初始化后对其进行适当的修改。

<!--more-->

# constinit 和 constexpr 有什么区别？

如果一个变量被声明为 `constexpr` ，那么它隐含地具有了 `constinit` 的性质。 `constexpr` 关键字用于指示编译器，在编译时可以计算该变量的值，并要求该变量在程序启动时被静态初始化。因此，`constexpr` 的变量天然地拥有 `constinit` 的特性
但是反过来并不成立，即如果一个变量被声明为 `constinit`，并不意味着它就是 `constexpr`。`constinit`仅确保变量在程序启动时被静态初始化，但不要求在编译时就能确定其值。相反，`constexpr` 要求在编译时就能确定变量的值。


```c++
#include <array>
constexpr std::size_t getArraySizeBasedOnArchitecture() {
    return sizeof(std::size_t) * 100;
}
constinit auto arrSize2 = getArraySizeBasedOnArchitecture();
constexpr auto arrSize1 = getArraySizeBasedOnArchitecture();
int main(){
     std::array<int, arrSize1> intArray1;  // Ok
     std::array<int, arrSize2> intArray2;  // compilation error
}
```

注意：**constexpr 和 constinit 不能同时出现在变量上。这会导致编译错误**。

# constinit 有什么作用？

- 保证变量在编译时初始化
- 当变量不能在编译时初始化时编译器生成错误提示
- 处理静态初始化顺序，[static-init-order](https://isocpp.org/wiki/faq/ctors#static-init-order) 

# 总结

- `constinit` 说明符仅适用于变量
- `constinit` 保证变量在编译过程中被初始化，否则我们会得到一个编译错误。
- `constinit` 说明符表示静态存储持续时间，但反之则不成立
- `constexpr` 变量意味着 `constinit` ，但反过来并不正确。
- 一个变量只能出现 `constexpr` 和 `constinit` 说明符之一。
- `constinit` 可以应用于 `const` 限定变量。