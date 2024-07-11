---
title: Qt QML 核心概念基础
date: 2024-02-26 09:50:56
tags:
- QML
- Qt
categories:
- Qt
---


# Qt QML 简介
## QML 是什么？

- `QML` 是声明式编程语言
- `QML` 模块 类型库
- 内置了 `javascript` 运行时环境, 提供逻辑处理： 界面逻辑，业务逻辑
## Qt Quick 是什么？

`Qt Quick` 是类型库，提供了可视化 `UI` 组件，软件开发框架，用于构建用户界面
<!--more-->
## QML 应用程序
## 使用 qmlscene 运行 QML 程序
## C++ 应用程序使用 QML
### 使用 QML Engine

只使用了 Qt Quick 框架，没有使用Qt Widgets

```c++
#include <QQmlApplicationEngine>

QQmlApplicationEngine engine;
engine.load(QUrl(QStringLiteral("qrc:/main.qml")));
if (engine.rootObjects().isEmpty())
    return -1;
```
### 使用 QML View

只使用了 Qt Quick 框架，没有使用 QWidget, QQuickView 不支持 window 作为根节点
```c++
#include <QQuickView>

const QUrl url(QStringLiteral("qrc:/main.qml"));
QQuickView* view = new QQuickView;
view->setSource(url);
view->show();
```
### 使用 QML Widget

支持混合使用Qt Quick 框架和 Qt Widgets 框架, 不支持 window 作为根节点

```c++
#include <QApplication>
#include <QQuickWidget>

QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
QApplication app(argc, argv);

const QUrl url(QStringLiteral("qrc:/main.qml"));
QQuickWidget *view = new QQuickWidget;
view->setSource(url);
view->show();
```

# Qt QML 语法基础

## QML 术语表
| 术语      | 含义 |
| ----------- | ----------- |
| QML      | 1. QML 是一种应用编程语言 2. QML模块实现了QML语言的架构和引擎       |
| Qt Quick   | 1 基于QML实现的类型和功能的标准库 2. Qt Quick 模块实现了这些类型和功能        |
| Type   | 1. QML 基础类型 2. QML对象类型        |
| Basic Type   | QML 内置类型。 int、bool、string等        |
| Object Type  | QML代码定义的对象类型 c++代码定义的对象类型        |
| Object   | 对象由QML引擎创建，1. 在对象定义时实例化 2.延迟实例化         |
| Component   | 组件是用于创建QML对象或者对象树的模板 1. QML文档加载时由QML引擎创建 2. 在QML 文档内部内联定义创建        |
| Document   | QML 文档包含了一些QML源代码，文档名以大写字母开头 1. 可以位于 QML 源代码文件 2. 可以位于一个文本字符串中        |
| Property   | 一个对象可以有一个或多个属性 1. 属性名称 2.属性的值        |
| Binding   | 属性绑定到一个javascript 表达式，任何时候属性的值由整个 javascript表达式估值决定        |
| Signal   | 对象可以发射一个信号  其他对象可以接受并通过信号处理器处理这个信号        |
| Signal Handler   | 信号处理        |
| Lazy Instance   | 对象可以延迟初始化，已避免不必要的工作        |
## QML 导入语句

- import 模块标识 主版本.次版本 as 模块名称
- import "目录路径" as 模块名称
- import "javascript文件名称" as 模块名称

## QML 对象定义语句

```
对象类型 {
    属性： 属性值
    信号
    信号处理器
    函数
}
```
## QML 注释
跟 `C++` 一样，单行和多行

# QML 调试

## 日志

```
console.log("宽度: ", width, "高度: ", height);
console.debug("宽度: ", width, "高度: ", height);
console.info("宽度: ", width, "高度: ", height);
console.warn("宽度: ", width, "高度: ", height);
console.error("宽度: ", width, "高度: ", height);
```
## 断言

**断言失败不影响之后的代码运行**

```
console.assert(width > 600, "assert failed")
```
## 计时器

```
console.time("ty");
...
console.timeEnd("ty");
```
## 跟踪
```
console.trace();
```
## 计数

统计函数执行了多少次

```
console.count();
```
