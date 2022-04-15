---
title: Effective C++ 26：尽可能推迟变量的定义
date: 2022-04-15 10:38:20
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 26:Postpone variable definitions as long as possible.

推迟变量的定义有两个好处：

- 改善程序效率，减少无用的构造和析构。
- 增加程序流程清晰度。

这条规则看似简单，但存在流程控制语句的时候容易疏忽。如：


```c++
string encryptPassword(const string& password){
    string encrypted;
    if (password.length() < MinimumPasswordLength) {
        throw logic_error("Password is too short");
    }
    encrypted = password;
    encrypt(encrypted);
    return encrypted;
}
```

## 推迟到需要构造时执行

当 `encryptPassword` 抛出异常时，`encrypted` 是无用的, 根本不需要构造它。所以更好的写法是推迟 `encrypted` 的构造：


```c++
string encryptPassword(const string& password){
    if (password.length() < MinimumPasswordLength) {
        throw logic_error("Password is too short");
    }
    string encrypted;       // 默认构造函数
    encrypted = password;   // 赋值运算符
    encrypt(encrypted);
    return encrypted;
}
```

## 推迟到有构造参数时

 **"尽可能延后"** 的真正意义。你不只应该延后变量的定义，直到非得使用该变量的前一刻为止，甚至**应该尝试延后这份定义直到能够给它初值实参为止**。如果这样，不仅能够避免构造(和析构)非必要对象，还可以避免无意义的 `default` 构造行为。

```c++
string encryptPassword(const string& password){
    if (password.length() < MinimumPasswordLength) {
       throw logic_error("Password is too short");
    }
    string encrypted(password);     // 拷贝构造函数
    encrypt(encrypted);
    return encrypted;
}
```

##  循环中的变量

循环中的变量定义也是一个常见的争论点。常有两种写法：

写法 `A`，在循环外定义：
 
```c++
Widget w;
for (int i = 0; i < n; ++i){ 
  w = some value dependent on i;
  ...                           
}                  

```

写法 `B` ，在循环内定义：

```c++
for (int i = 0; i < n; ++i) {
    Widget w(some value dependent on i);
    ...
}
```

- `A`：`1` 个构造函数，`1` 个析构函数，`n` 个赋值运算符
- `B`：`n` 个构造函数，`n` 个析构函数

但 `A` 使得循环内才使用的变量进入外部的作用域，不利于程序的理解和维护。软件工程中倾向于认为人的效率比机器的效率更加难得， 所以推荐采用 `B` 来实现。除非：

- 这段代码是性能的关键.
- 赋值比一对构造/析构更加廉价。