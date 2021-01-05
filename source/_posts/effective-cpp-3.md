---
title: Effective C++ 3：尽可能使用 const
date: 2021-01-05 22:04:32
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item3: Use const whenever possible.

# 常量的声明

指针的常量声明：

```c++
char greeting[] = "Hello";
char* p = greeting;                 //non-const pointer, non-const data
const char* p = greeting;           //non-const pointer, const data
char* const p = greeting;           //const pointer, non-const data
const char* const p = greeting;     //const pointer, const data
```
如果 `const` 出现在`*`左边，表示被指物为常量;　如果出现在`*`右边，表示指针自身为常量；如果出现在`*`两边，表示被指物和指针两者都是常量。

如果被指物是常量，`const` 放在类型之前和放在类型之后`*`之前表示的意义一样：

```c++
void f1(const Widget* p);　//f1　获得一个指针，指向一个常量Ｗidget对象
void f2(widget const *p);　//f2 也是
```


STL的`iterator` 系以指针塑模出来，所以`iterator`的作用像个`T*`指针。如果希望指针是常量，可以声明为 `const iterator`，如果希望被指物为常量，需使用 `const_iterator`

```c++
std::vector<int> vec;
...
const std::vector<int>::iterator iter = vec.begin();    //iter的作用像个Ｔ* const
*iter = 10;                                             //没问题，改变iter所指物  
++iter;　　　　　　　　　　　　　　　　　　　　　　 　　　　     //错误，iter是const
std::vector<int>::const_iterator cIter = vec.begin();   //cIter的作用像个const Ｔ*
*cIter = 10;                                            //错误，*cIter是const
++cIter;                                                //没问题，　改变cIter
```
返回值声明为常量，可以降低代码被错误使用:

```c++
class Rational　{...};
const Rational operator*{const Rational& lhs, const Rational& rhs};
```
当我们本来想做个比较，错误地输入`=`
```c++
if (a * b = c) ...
```
编译器就会报错误：不可给常量赋值。

# const 成员函数

声明const 成员函数，是为了确认该成员函数可以作用与const对象，也使class接口比较容易理解，可以得知哪些函数可以改动对象内容，哪些不可以。

成员函数只是常量性不同是可以被重载。

```c++
class TextBlock {
public:
  ...
  const char& operator[](std::size_t position) const   // operator[] for
  { return text[position]; }                           // const objects

  char& operator[](std::size_t position)               // operator[] for
  { return text[position]; }                           // non-const objects

private:
   std::string text;
};

TextBlock tb("Hello");
const TextBlock ctb("World");
tb[0] = 'x';             // fine — writing a non-const TextBlock
ctb[0] = 'x';            // error! — writing a const TextBlock
```

# bitsise constness 和　logical constness

`bitsise constness`: 成员函数只有在不改变对象的任何非静态成员变量时才可以被称为常量函数。也是C++对常量性的定义。

```c++
class TextBlock{
   
public:
    char& operator[](std::size_t position) const{
        return pText[position];
    }
private:
    char* pText;
};

const TextBlock tb;
char *p = &tb[1];
*p = 'a';
```



# 在const和non-const成员函数中避免重复

当`const`和`non-const`成员函数有着实质等价的实现时，令`non-const`函数调用`const`函数可以避免代码重复。不可以反着来。
```c++
const char& operator[](std::size_t position) const {
    ...
    return text[position]
}
char& operator[](std::size_t position) {
    return const_cast<char&>(
        static_cast<const TextBlock&>(*this)
            [position]
        )
}
```

1. `*this` 的类型是 `TextBlock`，先把它强制隐式转换为 `const TextBlock`，这样我们才能调用那个常量方法。
2. 调用 `operator[](std::size_t) const`，得到的返回值类型为 `const char&`。
3. 把返回值去掉 `const` 属性，得到类型为 `char&` 的返回值。