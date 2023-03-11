---
title: enable_shared_from_this的注意事项
date: 2023-03-11 10:09:57
tags:
- shared_ptr
- 智能指针
categories:
- C/C++
---

# 使用场景

当我们在对象函数中需要返回或者使用自己的 `shared_ptr` 指针时，该怎么办呢？常见的错误写法如下：用不安全的表达式试图获得 `this` 的 `shared_ptr` 对象, 但可能会导致 `this` 被多个互不知晓的所有者析构.

```c++
struct Bad
{
    std::shared_ptr<Bad> getptr() {
        return std::shared_ptr<Bad>(this);
    }
    ~Bad() { std::cout << "Bad::~Bad() called\n"; }
};
```

```c++
{
    std::shared_ptr<Bad> bp1 = std::make_shared<Bad>();
    std::shared_ptr<Bad> bp2 = bp1->getptr();
    std::cout << "bp2.use_count() = " << bp2.use_count() << '\n';
} // UB: Bad 对象将会被删除两次
```
正确写法是将定义对象公开继承 `enable_shared_from_this`: 
```c++
class Good: public std::enable_shared_from_this<Good> // 注意：继承
{
    std::shared_ptr<Good> getptr() {
        return shared_from_this();
    }
};
```

<!--more-->

`std::enable_shared_from_this` 能让其一个对象（假设其名为 `t` ，且已被一个 `std::shared_ptr` 对象 `pt` 管理）安全地生成其他额外的 `std::shared_ptr` 实例（假设名为 `pt1`, `pt2`, ... ） ，它们与 `pt` 共享对象 `t`的所有权。

# 实现

`enable_shared_from_this` 的常见实现为：其内部保存着一个对 `this` 的弱引用（例如 `std::weak_ptr` )。 `std::shared_ptr` 的构造函数发现是并能访问 `enable_shared_from_this` 的基类，并且若内部存储的弱引用未生成。则 `std::shared_ptr` 生成内部存储的弱引用。 `libc++` 的实现如下：

```c++
template<class _Tp>
class _LIBCPP_TEMPLATE_VIS enable_shared_from_this
{
    mutable weak_ptr<_Tp> __weak_this_;

    ...

public:
    _LIBCPP_INLINE_VISIBILITY
    shared_ptr<_Tp> shared_from_this()
        {return shared_ptr<_Tp>(__weak_this_);}
    _LIBCPP_INLINE_VISIBILITY
    shared_ptr<_Tp const> shared_from_this() const
        {return shared_ptr<const _Tp>(__weak_this_);}
    
    ...
}

```
`shared_ptr` 的构造函数：
```c++

    ...

    template<class _Yp, class = __enable_if_t<
        _And<
            __raw_pointer_compatible_with<_Yp, _Tp>
            // In C++03 we get errors when trying to do SFINAE with the
            // delete operator, so we always pretend that it's deletable.
            // The same happens on GCC.
#if !defined(_LIBCPP_CXX03_LANG) && !defined(_LIBCPP_COMPILER_GCC)
            , _If<is_array<_Tp>::value, __is_array_deletable<_Yp*>, __is_deletable<_Yp*> >
#endif
        >::value
    > >
    explicit shared_ptr(_Yp* __p) : __ptr_(__p) {
        unique_ptr<_Yp> __hold(__p);
        typedef typename __shared_ptr_default_allocator<_Yp>::type _AllocT;
        typedef __shared_ptr_pointer<_Yp*, __shared_ptr_default_delete<_Tp, _Yp>, _AllocT> _CntrlBlk;
        __cntrl_ = new _CntrlBlk(__p, __shared_ptr_default_delete<_Tp, _Yp>(), _AllocT());
        __hold.release();
        __enable_weak_this(__p, __p);
    }
    
    ...

 template <class _Yp, class _OrigPtr, class = __enable_if_t<
        is_convertible<_OrigPtr*, const enable_shared_from_this<_Yp>*>::value
    > >
    _LIBCPP_HIDE_FROM_ABI
    void __enable_weak_this(const enable_shared_from_this<_Yp>* __e, _OrigPtr* __ptr) _NOEXCEPT
    {
        typedef __remove_cv_t<_Yp> _RawYp;
        if (__e && __e->__weak_this_.expired())
        {
            __e->__weak_this_ = shared_ptr<_RawYp>(*this,
                const_cast<_RawYp*>(static_cast<const _Yp*>(__ptr)));
        }
    }
```

`c++17` 前对没初始化的 `weak_ptr` 的对象调用 `shared_from_this` 行为未定义行,  `C++17` 起抛出 `std::bad_weak_ptr` 异常：

```c++
    try {
        Good not_so_good;
        std::shared_ptr<Good> gp1 = not_so_good.getptr();
    } catch(std::bad_weak_ptr& e) {
        // C++17 前为未定义行为； C++17 起抛出 std::bad_weak_ptr 异常
        std::cout << e.what() << '\n';    
    }
```

# 相关问题

## 什么时候初始化 `enable_shared_from_this` 中的 `weak_ptr`？

参考实现部分，构造 `shared_ptr` 的对象时判断是 `enable_shared_from_this`的时候会初始化。

## 能不能非公有继承 `enable_shared_from_this` ？

**不能**

非公有继承时候，判断 `class` 是否是 `enable_shared_from_this` 会失败就不会去初始化 `weak_ptr`。

## 能不能在构造函数中调用？

**不能**

`shared_ptr` 初始化需要调用构造函数，而 `weak_ptr` 需要 `shared_ptr` 构造初始化。`GG` 了。

# 参考资料

* [std::enable_shared_from_this](https://en.cppreference.com/w/cpp/memory/enable_shared_from_this)
