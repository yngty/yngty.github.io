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

例如：

```make
hello.o: hello.c
	gcc -c hello.c -o hello.o
```
含义：当 hello.c 更新时，执行 gcc -c hello.c -o hello.o 生成 hello.o。

> make 会自动根据文件的时间戳判断哪些目标需要重新生成，这就是它的高效之处。

<!--more-->

# 三、自动变量

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

### 示例 1：`$@` 和 `$<`

```make
hello.o: hello.c
	gcc -c $< -o $@
```
等价于:

```make
hello.o: hello.c
	gcc -c hello.c -o hello.o
```
### 示例 2：`$^`（所有依赖）

```make
main: main.o utils.o
	gcc $^ -o $@
```

等价于：

```make
main: main.o utils.o
	gcc main.o utils.o -o main
```

### 示例 3：模式规则 `%`

```make
%.o: %.c
	gcc -c $< -o $@
```

这个模式规则可以自动匹配：

- `main.o ← main.c`
- `utils.o ← utils.c`

此时

- `$*` 表示不带扩展名的 `stem`，比如 `main`、`utils`
- `$<` 是 `main.c`
- `$@` 是 `main.o`

### 示例 4：`$?`（比目标新的依赖）

```make
backup.tar: file1 file2
	tar -cf $@ $?
```

含义：
- `backup.tar` 是目标
- `$?` 是所有比 `backup.tar` 新的文件（需要重新打包的部分）

# 四、wildcard / patsubst / basename
除了自动变量，`Makefile` 还提供了强大的字符串处理函数，常见的有 `wildcard`、`patsubst` 和 `basename`。
它们可以帮助你实现自动化构建，让 `Makefile` 更加简洁和可扩展。


## 1. wildcard —— 查找文件
语法：

```make
$(wildcard pattern...)
```
- PATTERN：文件匹配模式（支持 *、?、[...] 等 shell 风格的通配符）

- 返回值：匹配到的文件名列表（以空格分隔）

> 💡 简单来说，`wildcard` 会去 磁盘 上找文件，不是对字符串做替换，而是 真实地查找存在的文件。

示例：

```make
SRC := $(wildcard *.c)

all:
	@echo $(SRC)
```

如果目录中有：

```css
main.c foo.c bar.c readme.txt
```
输出：

```css
main.c foo.c bar.c
```

💡 **作用**：自动扫描匹配的文件，而不是手写文件名。

## 2. patsubst —— 模式替换
语法：

```make
$(patsubst pattern,replacement,text)
```

- <pattern>：要匹配的模式（支持 `%` 通配符）

- <replacement>：替换成的新模式（同样支持 `%`）

- <text>：要进行替换的一组字符串（以空格分隔）


示例：

```make

SRC := foo.c bar.c main.c
OBJ := $(patsubst %.c,%.o,$(SRC))

all:
	@echo $(OBJ)
```
输出：

```css
foo.o bar.o main.o
```

💡 **作用**：批量替换文件名后缀，例如 .c → .o。

## 3. basename —— 去掉扩展名
语法：

```make
$(basename names...)
```
示例：

```make
FILES := foo.c bar.c readme.txt
NAMES := $(basename $(FILES))

all:
	@echo $(NAMES)
```
输出：

```
foo bar readme
```
💡 **作用**：获取不带扩展名的“纯文件名”。

## 4. 三者结合的自动化构建
假设目录中有：

```
bootstrap.bpf.c bootstrap.c
user_ringbuf.bpf.c user_ringbuf.c
```

我们想要：

自动找到所有 `.bpf.c` 文件

自动去掉扩展名

自动生成 `.bpf.o` 和 `.skel.h` 列表

```make

PROGRAMS := $(basename $(wildcard *.bpf.c))
BPF_OBJS := $(patsubst %,%.bpf.o,$(PROGRAMS))
SKELS    := $(patsubst %,%.skel.h,$(PROGRAMS))

all:
	@echo "PROGRAMS: $(PROGRAMS)"
	@echo "BPF_OBJS: $(BPF_OBJS)"
	@echo "SKELS: $(SKELS)"
```

执行结果：
```makefile
PROGRAMS: bootstrap user_ringbuf
BPF_OBJS: bootstrap.bpf.o user_ringbuf.bpf.o
SKELS: bootstrap.skel.h user_ringbuf.skel.h
```

新增一个 `trace_connect.bpf.c` 后，`Makefile` 会自动识别并更新，无需修改变量。
