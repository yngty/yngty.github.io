---
title: Effective-STL 9：慎重选择删除元素的方法
date: 2023-02-16 15:11:25
tags:
- Effective-STL
- C++
- stl
categories:
- Effective-STL
---

> Item9. Choose carefully among easing options.

# 一、删除特定值

1. `vector`、 `string` 或 `deque`

    最好使用 `erase-remove`习惯用法: 

    ```c++
    c.erase(remove(c.begin(), c.end(), 1963, c.end()));
    ```
2. `list`

    直接使用 `remove` 方法:

    ```c++
    c.remove(1963);
    ```
3. 标准关联容器

    直接使用 `erase` 方法:
    ```c++
    c.erase(1963)
    ```
<!--more-->
# 二、删除满足特定判定条件的值

```c++
bool badValue(int) { return true; } // 返回x是否为"坏值"
```

1. 对于 `vector`、 `string` 或 `deque`

    ```c++
    c.erase(remove_if(c.begin(), c.end(), badValue, c.end()));
    ```
2. 对于 `list` 容器

    ```c++
    c.remove_if(badValue);
    ```

3. 对于标准关联容器

    **把当前的i传给erase，i后缀递增**

    ```c++
    for (AssocContainer<int>::iterator i = c.begin(); i != c.end();) {
        if (badValue(*i)) c3.erase(i++); 
        else ++i;                     
    }
    ```

# 三、循环内部删除对象之外还要做某些事

```c++
void doSomething(int) { ... }
```
1. `vector`、 `string` 或 `deque`

    **接收 `erase`返回的迭代器值。**
    ```c++
    for (SeqContainer<int>::iterator i = c.begin(); i != c.end();) {
        if(badValue(*i)) {
            doSomething(*i);
            i = c.rease(i);
        } else ++i;
    }
    ```
2. 对于 `list` 容器

    虽然也可以采用标准关联容器方法，但建议采用跟 `vector`、 `string` 或 `deque` 一致。

3. 对于标准关联容器

    ```c++
    for (SeqContainer<int>::iterator i = c.begin(); i != c.end();) {
        if(badValue(*i)) {
            doSomething(*i);
            c.rease(i++);
        } else ++i;
    }
    ```