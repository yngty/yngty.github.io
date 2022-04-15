---
title: Effective C++ 25：设计一个不抛异常的 swap 函数
date: 2022-04-09 23:12:46
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Consider support for a non-throwing swap.


`swap` 函数能置换两对象值，功能很重要!
- 异常安全性编程
- 处理自我赋值可能性：{% post_link effecitve-cpp/effective-cpp-11 赋值运算符需要考虑自我赋值问题 %}

`std` 的缺省基本实现如下：

```c++
namespace std {
    template <typename T>
    void swap(T& a, T& b) {
        T temp(a);
        a = b;
        b = temp;
    }
}

<!--more-->

```

## 类的 swap

只要类型 `T` 支持 `copying`运算(拷贝构造和拷贝赋值运算)就能使用。 但缺省实现会有多次拷贝，在某些情况下不是性能最好的实现。比如针对 `pimpl` 手法实现的 `class`, 不仅要复制三次 `Widget` 还需要复制三次 `WdigetImpl`, 非常缺乏效率。

```c++
class WidgetImpl {
public:
    ...

private:
    int a, b, c;
    std::vector<double> v;
    ...
};

class Widget {

public:
    Widget(const Widget&);

    Widget& operator= (const Widget& rhs) {
        ...
        *pImpl = *(rhs.pImpl);
        ...

    }
private:
    WidgetImpl *pImpl;
};
```

其实我们发现这种情况只需要将 `pImpl` 指针交换就好， 我们可以将 `std::swap` 对 `Widget` 的特化来实现.

```c++
namespace std {
    template <>
    void swap<Widget> (Widget& a, Widget& b) {
        swap(a.pImpl, b.pImpl);
    }
}
```
但上述代码不能通过编译， 因为 `pImpl` 是私有变量， 所以，`Widget` 应当提供一个 `swap` 成员函数或友元函数。 惯例上会提供一个成员函数：

```c++
class Widget {

public:
    ...
    
    void swap(Widget& other) {
        using std::swap; // 为何要这样？请看下文
        swap(pImpl, other.pImpl);
    }

    ...
};

namespace std {
  template<>
  void swap<Widget>(Widget& a, Widget& b){
      a.swap(b);              // 调用成员函数
  }
}
```
上述实现与 STL 容器是一致的：**提供公有 `swap` 成员函数， 并特化 `std::swap` 来调用那个成员函数**。


## 类模板的 swap

如果 `Widget` 和 `WidgetImpl` 是 `class templates` 而非 `classes`, 按照上面的 `swap` 实现方式，你可能会这样写：

```c++
template<typename T>
class Widget{  ... };

template<typename T>
class WidgetImpl{ ... };

namespace std {
    template<typename T>
    void swap<Widget<T>>(Widget<T>& a, Widget<T>& b) {
        a.swap(b);
    }
}
```

但上述代码不能通过编译， `c++` 允许偏特化类模版，却不允许偏特化函数模版(虽然有的编译器中可以编译)。那我们继续尝试重载 `std::swap`  函数：

```c++
namespace std{
    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b) {
        return a.swap(b);
    }
}
```

这里我们重载了 `std::swap`，相当于在 `std` 命名空间添加了一个函数模板。但这在 `C++` 标准中是不允许的！ `C++` 标准中，客户只能特化 `std` 中的模板，但不允许在 `std` 命名空间中添加任何新的模板。 上述代码虽然在有些编译器中可以编译，但会引发未定义的行为，所以不要这么做。所以我们最终可以把 `swap` 定义在 `Widget` 所在的命名空间中：

```c++ 
namespace WidgetStuff {

    template<typename T> 
    class Widget { ... };

    template <typename T>
    void swap(Widget<T>& a, Widget<T>& b) {
        return a.swap(b);
    }
}
```

任何地方在两个 `Widget` 上调用 `swap` 时，`C++` 根据其 `argument-dependent lookup`（又称 `Koenig lookup`） 会找到 `WidgetStuff` 命名空间下的具有 `Widget` 参数的 `swap`。

其实类的 `swap` 也可以在同一命名空间下定义 `swap` 函数，而不必特化 `std::swap`。 但有人可能直接写 `std::swap(w1, w2)`，特化 `std::swap` 可以让你的类更加健壮。

**在成员函数中不要直接调用 `swap(pImpl, other.pImpl);` 因为指定了调用 `std::swap`，`argument-dependent lookup` 便失效了，`WidgetStuff::swap` 不会得到调用**。

如果希望优先调用 `WidgetStuff::swap`，如果未定义则取调用 `std::swap`，那么应该如何写呢？ 看代码：

```c++
template<typename T>
void doSomething(T& obj1, T& obj2){
  using std::swap;           // 使得 std::swap 在该作用域内可见
  swap(obj1, obj2);          // 现在，编译器会帮你选最好的 swap
}
```

此时，`C++` 编译器还是会优先调用指定了 `T` 的 `std::swap`，其次是 `obj1` 的类型 `T` 所在命名空间下的对应 `swap` 函数， 最后才会匹配 `std::swap` 的默认实现。

## 总结
如何实现 `swap` 呢？

- 提供一个更加高效的，不抛异常的公有成员函数（比如 `Widget::swap`）。
- 在你类（或类模板）的同一命名空间下提供非成员函数 `swap`，调用你的成员函数。
- 如果你写的是类而不是类模板，也可以特化 `std::swap`，同样地在里面调用你的成员函数。
- 调用时，请首先用 `using` 使 `std::swap` 可见，然后直接调用 `swap`。