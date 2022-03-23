---
title: Effective C++ 20：传常量引用比传值更好
date: 2021-11-11 16:36:55
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 20: Prefer pass-by-reference-to-const to pass-by-value.

缺省情况下`C++` 用传值得方式(一个继承自`C`的方式)传递对象至(或来自)函数。除非你另外指定，否则函数参数都是以实际实参的复件(副本)为初值，而调用端所获得的亦是函数返回值的一个复件。这些复件(副本)系由对象的`copy`构造函数产出。

**尽量以传常量引用替换传值前者通常比较高效，并可避免切割问题 (`slicing problem`)，但是内置类型和 `STL` 迭代器，还是传值更加合适。**。


## 性能问题:

```c++
class Person {
public:
    Person ();
    virtual -Person();
private:
    std::string name;
    std::string address;
}

class Student: public Person {
public:
    Student();
    -Student();
private:
    std::string schoolName;
}
```

现在考虑以下代码，其中调用函数 `validateStudent` ，后者需要一个 `Student`
(`by value`) 并返回它是否有效:

```c++
bool validateStudent(Student s);           // function taking a Student by value

Student plato;                             // Plato studied under Socrates
bool platoIsOK = validateStudent(plato);   // call the functio
```

在调用 `validateStudent(`) 时进行了 **6** 个函数调用：

1. `Person` 的拷贝构造函数，为什么 `Student` 的拷贝构造一定要调用 `Person` 的拷贝构造请参见：{% post_link effecitve-cpp/effective-cpp-12 Item:12 复制对象时勿忘其每一个成分 %}
2. `Student` 的拷贝构造函数
3. `name`, `address`, `schoolName`, `schoolAddress` 的拷贝构造函数

解决办法便是传递常量引用：

```c++
bool validateStudent(const Student& s);
```

首先以引用的方式传递，不会构造新的对象，避免了上述例子中 **6** 个构造函数的调用。 同时 const 也是必须的：传值的方式保证了该函数调用不会改变原来的 `Student`， 而传递引用后为了达到同样的效果，需要使用 `const` 声明来声明这一点，让编译器去进行检查!

## 截断问题

```c++
class Window {
public:
...
std::string name() const;           // return name of window
virtual void display() const;       // draw window and contents
};

class WindowWithScrollBars: public Window {
public:
...
virtual void display() const;
};
```

现在假设你希望写个函数打印窗口名称，然后显示该窗口:

```c++
void printNameAndDisplay(Window w)
    std::cout << w.name();
    w.display() ;
}

WindowWithScrollBars wwsb;
printNameAndDisplay(wwsb);
```

当调用 `printNameAndDisplay` 时参数类型从 `WindowWithScrollBars` 被隐式转换为 `Window`。 该转换过程通过调用 `Window` 的拷贝构造函数来进行。 导致的结果便是函数中的 `w` 事实上是一个 `Window` 对象， 并不会调用多态子类 `WindowWithScrollBars` 的 `display()`。

正确做法：

```c++
// fine, parameter won't be sliced
void printNameAndDisplay(const Window& w){ 
    std::cout << w.name();
    w.display();
}
```

## 特殊情况

一般情况下相比于传递值，传递常量引用是更好的选择。但也有例外情况，比如 **内置类型** 和 **STL 迭代器**和**函数对象**。