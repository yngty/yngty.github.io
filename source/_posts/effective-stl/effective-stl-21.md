---
title: Effective-STL 21：总是让比较函数在等值情况下返回 false
date: 2023-03-10 21:05:20
tags:
- Effective-STL
- C++
- stl
categories:
- Effective-STL
---

> Item 21: Always have comparison functions return false for equal values.


# 严格弱序( `strict weak ordering` )

先补充下严格弱序的概念: 对两个变量 `x` 和 `y`：

- `x > y` 等同于 `y < x`
- `x == y` 等同于 `!(x < y) && !(x > y)`

要想严格弱序，就需要遵循如下规则：

- 每个变量值必须等于其本身（`irreflexivity`）：`x < x` 永远不能为 `true`
- 不对称性（`asymmetry`）：如果 `x < y`，那么 `y < x` 就不能为 `true`
- 有序性必须可传递性：如果 `x < y` 并且 `y < z`，那么 `x < z`
- 值相同必须具有可传递性：如果 `x == y` 并且 `y == z`，那么 `x == z`

<!--more-->

# 为什么？

## 1. 关联容器中的比较算法

比如我们创建一个 `set` ， 用 `less_euqal` 作为比较类型，然后插入两个 `10` ：

```c++
set<int, less_equal<int>> s;
s.insert(10);
s.insert(10);
```

我们将第一个 `10` 记为 `10A`, 第二个 `10` 记为 `10B`, 我们在插入 `10B` 的时候会检查是否与 `10A` 相同, 我们用的是 `less_equal`，下面的表达式会为假，就会重复插入，显然不合理。:
```
!(10A <= 10B) && !(10B <= 10A) //  !(true) && !(true)
```

另外在 `multiset` 中也不行: 

```c++
multiset<int, less_equal<int>> s;
s.insert(10); // 插入10A
s.insert(10); // 插入10B
```

当我们想要一个 `equal_range`, `10A` 和 `10B` 同样认为是不等，永远不会在同一个区间。

## 2. `sort` 算法

对于 `std::sort`，当容器里面元素个数大于 `_S_threshold` 的值时（`16`），就会使用快速排序，会将所有的元素与中间值比较是无边界保护的，实现如下：

```c++
template<typename _RandomAccessIterator, typename _Tp, typename _Compare>
     _RandomAccessIterator
     __unguarded_partition(_RandomAccessIterator __first,
               _RandomAccessIterator __last,
               _Tp __pivot, _Compare __comp)
{
    while (true)
    {
       while (__comp(*__first, __pivot)) // <-------------------
         ++__first;
       --__last;
       while (__comp(__pivot, *__last))
         --__last;
       if (!(__first < __last))
         return __first;
       std::iter_swap(__first, __last);
       ++__first;
    }
}
```

如果传入的 `vector` 中，后面的元素完全相等， `__comp()`函数一直返回 `true` ，在进行快速排序的时候，`++first` 就可能越界失效，导致 `coredump`。
