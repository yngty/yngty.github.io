---
title: Effective C++ 2：尽量以const, enum, inline 替换 &#35;define
date: 2020-12-14 12:36:49
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item 2: Prefer consts, enums, and inlines to #defines

我们先看看`#deifne` 有哪些的问题:

# 不利于调试

```
#define ASPECT_RATION 1.653
```
在预处理时候 `ASPECT_RATION` 可能就被移走了,`ASPECT_RATION` 没有进入 符号表, 运行此常量获得编译错误信息时, 可能会疑惑。因为这个错误信息总是提到 `1.653`，而不是`ASPECT_RATION` ， 如果 `ASPECT_RATION` 定义不是自己写的头文件中，可能对 `1.653` 的来源毫无概念，将因追踪它浪费时间，解决之道是以一个常量替换上述宏 。
```c++
const double AspectRatio = 1.653 //大写名称通常用于宏
                                 //因此这里改变名称写法
```
作为一个语言常量，`ASPECT_RATION` 肯定会被编译器看到，当然会进入记号表内。此外对于浮点常量(`floating point constant`)而言，使用常量可能比使用`#define` 导致较少量的码。


# 不重视scope
无法利用 `#define` 创建`class`专属常量。一旦宏定义，它就在其后的编译过程中有效（除非在某处 `#undef` ）。而 `const` 可以。

```c++
class GamePlayer {
private:
    static const int NumTurns; //常量声明式
    int scores[NumTurns];      //使用该常量

}
```

## enum 比 const 更好用
旧式编译器也许不支持上述语法，　它们不允许static在声明式上获得初值，此外所谓的“`in-classs　初值设定`”也只运行对**整数常量**进行，　如果编译器不支持上述语法，可以将初值放在定义式

```c++
class CostEstimate {
public:
    static const double FudgeFactor;  //staitc class　常量声明位于头文件内
}
const double CostEstimate::FudgeFactor = 1.35; //staitc class　常量定义位于实现文件内
```
如果使用`emnu`就很简单：
```c++
class GamePlayer {
private:
    enum { NumTurns = 5 };

    int scores[NumTurns];　//the enum hack
}
```
　
# 不易理解

```c++
#define CALL_WITH_MAX(a, b)  f((a) > (b) ? (a) : (b))

int a = 5, b =0;
CALL_WITH_MAX(++a, b);     　//ａ被累加二次
CALL_WITH_MAX(++a, b + 10);　//ａ被累加一次
```

- 必须记住为宏的所有实参加上小括号
- 在这里调用ｆ之前，ａ的递增次取决与“它被拿来与谁比较”

更好的做法是使用　`template inline`　函数。

```c++
template <typename T>
inline void callWithMax(const T &a, const T &b)
{
    return (a > b ? a : b);
}

```


