---
title: Makefile 拾遗
date: 2025-07-31 11:18:39
tags:
- Makefile
---

## 一、什么是 Makefile

`Makefile` 是为 `make` 命令准备的脚本文件，它描述了项目中文件之间的依赖关系以及如何编译它们。它允许你只输入一条命令 `make`，即可自动编译项目中所有需要更新的部分。

## 二、Makefile 基本语法

最基本的格式如下：

```make
目标: 依赖
	命令
```
<!--more-->

例如：

```make
hello.o: hello.c
	gcc -c hello.c -o hello.o
```
这表示当 `hello.c` 更新时，使用 `gcc` 编译生成 `hello.o`。

# 自动变量

自动变量是 `make` 在执行规则时自动设置的特殊变量。它们根据当前规则中的目标和依赖动态赋值，不需要你手动传参。

| 自动变量 | 含义                             |
|----------|----------------------------------|
| `$@`     | 当前规则的 **目标（Target）**    |
| `$<`     | 当前规则的 **第一个依赖**        |
| `$^`     | 当前规则的 **所有依赖（去重）**  |
| `$+`     | 当前规则的 **所有依赖（保留重复）** |
| `$*`     | 去掉后缀的文件名（stem）         |
| `$?`     | 所有**比目标新的依赖项**         |

## 示例讲解

### 示例 1：用 `$@` 和 `$<`

```make
hello.o: hello.c
	gcc -c $< -o $@
```
等价于:

```make
hello.o: hello.c
	gcc -c hello.c -o hello.o
```
### 示例 2：多个依赖用 `$^`

```make
main: main.o utils.o
	gcc $^ -o $@
```

等价于：

```make
main: main.o utils.o
	gcc main.o utils.o -o main
```

### 示例 3：用 `$*` 构建 `.c` 和 `.o`

```make
%.o: %.c
	gcc -c $< -o $@
```

这个模式规则可以自动匹配：

- `main.o ← main.c`
- `utils.o ← utils.c`
- `$*` 表示不带扩展名的 `stem`，比如 `main`、`utils`
- `$<` 是 `main.c`
- `$@` 是 `main.o`

### 示例 4：使用 `$?` 构建增量目标

```make
backup.tar: file1 file2
	tar -cf $@ $?
```

含义：
- `backup.tar` 是目标
- `file1` `file2` 是依赖
- `$?` 表示比 `backup.tar` 新的文件（即需要重新打包的）
