---
title: Effective C++ 28：避免返回 handles 指向对象内部成分
date: 2022-04-16 21:42:47
tags:
- Effective-C++
- C++
categories:
- Effective-C++
---

> Item28: Avoid returning "handles" to object internals.

避免返回 `handles` (包括 `references` 、指针、迭代器)指向对象内部。

<!--more-->

## 破坏封装性

`const` 函数不再是 `const`, 修改了私有成员变量。

```c++
class Point {
public:
    Point(int x, int y);
    ...
    void setX(int x);
    void setY(int y);
    ...
};

struct RectData {
    Point ulhc;
    Point lrhc;
};

class Rectangle {
    ...
    Point& upperLeft() const { return pData->ulhc; }
    Point& lowerRight() const { return pData->lrhc; }
    ...
private:
    std::shared_ptr<RectData> pData;
}

```
虽然这样的设计可通过编译，但却是错误的。`upperLeft` 和 `lowerRight` 被声明为 `const` 成员函数，但是可以更改内部数据。

```c++
Point coord1(0, 0);
Point coord2(100, 100); 
const Rectangle rec(coord1, coord2); // rec是个const矩形, 从 (0 ，0) 到 (100 ， 100)

rec.upperLeft( ) .setX(50);  // 现在rec却变成从 (50 ， 0) 到 (100 ， 100)
```

- 成员变量的封装性最多只等于"返回其 `reference`" 的函数的访问级别。
- 如果 `const` 成员函数传出一个 `reference`，后者所指数据与对象自身有关联，而它又被存储于对象之外，那么这个函数的调用者可以修改那笔数据。(`bitwise constness`原因)

## 悬空问题

虽然我们可以修改函数，达到不能修改私有成员变量。
```c++
const Point& upperLeft() const { return pData->ulhc; }
```
但也不能解决悬空问题。如下：

```c++
class GUIObject { ... };
const Rectangle boundingBox(const GUIObject& obj); //以 by value 方式返回一个矩形
```

现在，客户有可能这么使用这个函数:

```c++
GUIObject* pgo; // 让pgo指向某个GUIObject
...

const Point* pUpperLeft = &(boundingBox(*pgo) .upperLeft()); // 取得一个指针指向外框左上点
```

`pUpperLeft` 被悬空了，`boundingBox(*pgo)` 返回的是一个临时变量，在语句执行结束后就会销毁，导致 `pUpperLeft` 指针失效。