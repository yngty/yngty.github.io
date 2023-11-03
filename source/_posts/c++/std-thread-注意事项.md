---
title: std::thread 注意事项
date: 2023-11-03 14:35:22
tags:
- thread
categories:
- C/C++
---

# `join` 和 `detach`

- `join` 或者 `detach` 只能调用**一次**

  当调用 `join` 或者 `detach` 之后会将持有的线程ID置为 `0`, 再次调用会抛异常。

  ```c++
  void
  thread::join()
  {
      int ec = EINVAL;
      if (!__libcpp_thread_isnull(&__t_))
      {
          ec = __libcpp_thread_join(&__t_);
          if (ec == 0)
              __t_ = _LIBCPP_NULL_THREAD;
      }

      if (ec)
          __throw_system_error(ec, "thread::join failed");
  }

  void
  thread::detach()
  {
      int ec = EINVAL;
      if (!__libcpp_thread_isnull(&__t_))
      {
          ec = __libcpp_thread_detach(&__t_);
          if (ec == 0)
              __t_ = _LIBCPP_NULL_THREAD;
      }

      if (ec)
          __throw_system_error(ec, "thread::detach failed");
  }
  ```
<!--more-->
- `thread` **不能拷贝只能移动**，但只能移动到没绑定线程的 `thread`。

```c++
class _LIBCPP_EXPORTED_FROM_ABI thread
{
    __libcpp_thread_t __t_;

    thread(const thread&);
    thread& operator=(const thread&);
     _LIBCPP_INLINE_VISIBILITY
    thread& operator=(thread&& __t) _NOEXCEPT {
        if (!__libcpp_thread_isnull(&__t_))
            terminate();
        __t_ = __t.__t_;
        __t.__t_ = _LIBCPP_NULL_THREAD;
        return *this;
    }
    ...
};
```
# 线程析构

通过 `std::thread` 创建的线程**必须调用 `join` 或者 `detach`**。线程析构时会先判断线程ID为 `0` 就抛异常。

```c++
thread::~thread()
{
    if (!__libcpp_thread_isnull(&__t_))
        terminate();
}
```

`std::thread` 析构默认不会等待线程结束，在 `C++20` 可以使用 `std::jthread`, `std::jthread` 实际是使用了 `RAII` 技术， 在内部持有 `thread` 成员变量，在析构时调用 `join` 函数。

```C++
class _LIBCPP_AVAILABILITY_SYNC jthread {

...

_LIBCPP_HIDE_FROM_ABI ~jthread() {
    if (joinable()) {
        request_stop();
        join();
    }
}
thread __thread_;
};
```