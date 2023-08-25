---
title: libevent 使用入门
date: 2023-08-23 10:07:43
tags:
- libevent
categories:
- Network
---

# 概述
`libEvent`, 一个事件通知库。有以下特点：

- 事件驱动，高性能；
- 轻量级，专注于网络；
- 跨平台，支持 `Windows`、`Linux`、`Mac` 等；
- 支持多种 `I/O` 多路复用技术， `epoll`、`poll`、`dev/poll`、`select` 和 `kqueue` 等；
- 支持 `I/O` ，定时器和信号等事件；

# libevent接口分析

libevent 接口大概分为以下几类: 

- 环境配置和初始化
    - `event_base_new`
- `evutil socket` 函数封装
    - `evutil_make_socket_nonblocking`
    - `evutil_make_listen_socket_reuseable`
        - set SO_REUSEADDR on Unix and does nothing on Windows
    - `evutil_closesocket`
- 事件`IO`处理
    - `event_new`
- 缓冲`IO`
    - `bufferevent`
- 循环(`Loop`)
    - `event_base_dispatch`

# Libevent API

## libevent上下文创建

- event_base *event_base_new(void)
- event_base *event_base_new_with_config( const struct event_config *);
    - 配置参数
        - event_config *event_config_new(void);
        - void event_config_free(struct event_config * cfg);
- event_reinit
    - int event_reinit(struct event_base *base); 调用 fork 之后可以正确工作
- void event_base_free(struct event_base *); 释放event_base内部分配的空间及其本身对象的空间，不释放事件和socket和在回调函数中申请 的空间
- event_config_set_flag
- event_config_avoid_method(struct event_ config *cfg, const char *method);
- event_config_require_features

event_base_config_flag

- EVENT_BASE_FLAG_NOLOCK: 
    - 不要为event_base 分配锁。设置这个选项可以为event_base 节省一点用于锁定和解锁的时间，但是让在多个线程中访问event_base 成为不安全的。

- EVENT_BASE_FLAG_IGNORE_ENV
    - 选择使用的后端时，不要检测EVENT_*环境变量。使用这个标志需要三思:这会让用户更难 调试你的程序与libevent 的交互。
- EVENT_BASE_FLAG_STARTUP_IOCP
    - 仅用于Windows,启用任何必需的IOCP 分发逻辑
    - iocp
        - event_config_set_num_cpus_hint
        - event_config_set_flag(cfg, EVENT_BASE_ FLAG_STARTUP_IOCP)
        - evthread_use_windows_threads();
- EVENT_BASE_FLAG_NO_CACHE_TIME
    - 不是在事件循环每次准备执行超时回调时检测当前时间，而是在每次超时回调后进行检 测。注意:这会消耗更多的CPU 时间。
- EVENT_BASE_FLAG_EPOLL_USE_ CHANGELIST
    - epoll下有效，防止同一个fd多次激发事件，fd如果做复制会有bug
- EVENT_BASE_FLAG_PRECISE_TIMER
    - 默认使用系统最快的记时机制，如果系统有较慢 且更精确的则采用


event_method_feature

- EV_FEATURE_ET = 0x01
    - 边沿触发的后端
- EV_FEATURE_O1 = 0x02,
    - 要求添加、删除单个事件，或者确定哪个事件激 活的操作是O(1)复杂度的后端
- EV_FEATURE_FDS = 0x04,
    - 要求支持任意文件描述符，而不仅仅是套接字的 后端
- EV_FEATURE_EARLY_CLOSE = 0x08
    - 检测连接关闭事件。您可以使用它来检测连接何时关闭，而不必从连接中读取所有挂起的数据。 并非所有后端都支持EV_CLOSED。允许您使用EV_CLOSED，而不需要读取所有挂起的数据。无法在所有内核版本上

## 事件Event处理

## 循环(loop)

## 缓冲 bufferevent

## libevent http