---
title: CMake 踩坑记
date: 2023-04-07 10:26:10
tags:
- CMake
---

# `install(CODE)` 和 `execute_process` 配合

## 确保 `WORKING_DIRECTORY` 存在

示例代码: 
```cmake
set(CMAKE_INSTALL_PREFIX ${PROJECT_BINARY_DIR}/export)
install(CODE "
    execute_process(COMMAND ${CMAKE_COMMAND} -E make_directory folder 
    WORKING_DIRECTORY \${CMAKE_INSTALL_PREFIX})
    ") 
```

注意确保 `CMAKE_INSTALL_PREFIX` 存在，可能执行这段代码时还没有 `install target` 导致 `CMAKE_INSTALL_PREFIX` 还没有生成。
<!--more-->

## 注意变量转义
 
**不转义容易在 `package` 时，执行 `execute_process` 因为路径问题出错**。

示例代码: 
```cmake
set(CMAKE_INSTALL_PREFIX ${PROJECT_BINARY_DIR}/export)
install(CODE "
    execute_process(COMMAND ${CMAKE_COMMAND} -E make_directory folder
    WORKING_DIRECTORY \${CMAKE_INSTALL_PREFIX})
    ") 
```
注意 `${WORKING_DIRECTORY}` 之前是有 `\` 转义，当没有转义时，`CMAKE_INSTALL_PREFIX`会直接替换，此时 `CMAKE_INSTALL_PREFIX` 表示的 `install` 路径。 `make package`时会到一个临时目录处理，其中的 `CMAKE_INSTALL_PREFIX` 跟  `install` 路径是不一样的，所以执行 `execute_process` 会出问题。当转义了表示 execute_process时再去获取 `CMAKE_INSTALL_PREFIX`, 这时能获取到正确路径。可以在 `cmake_install.cmake` 查看生成的代码。注意 `WORKING_DIRECTORY` 的值。

### 不转义生成代码：
```cmake 
if(CMAKE_INSTALL_COMPONENT STREQUAL "rel" OR NOT CMAKE_INSTALL_COMPONENT)
    execute_process(COMMAND /Users/xxx/Qt/Tools/CMake/CMake.app/Contents/bin/cmake -E make_directory folder
    WORKING_DIRECTORY "/xxx/xxx/xxxx/export")
endif()
```

转义生成代码：

```cmake 
if(CMAKE_INSTALL_COMPONENT STREQUAL "rel" OR NOT CMAKE_INSTALL_COMPONENT)
    execute_process(COMMAND /Users/xxx/Qt/Tools/CMake/CMake.app/Contents/bin/cmake -E make_directory folder
    WORKING_DIRECTORY ${CMAKE_INSTALL_PREFIX})
endif()
```

# 在 `MacOS` 上 `No CMAKE_CXX_COMPILER could be found`

```shell
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer 
```

# CMAKE_OSX_ARCHITECTURES

- 设置 `macOS`和 `iOS` 的特定架构
- 应在第一次 `project()` 或 `enable_language()` 命令之前设置
- 应设置为 `CACHE` 条目, 除非策略 `CMP0126` 设置为 `NEW` 
- 在 `Apple` 以外的平台上被忽略

```
if (APPLE)
    set(CMAKE_OSX_ARCHITECTURES x86_64 CACHE STRING "")
endif()
```

# cmakedefine 的使用例子

`#cmakedefine` 用于 `configure_file()` 中用于生成头文件的文件中，只有当 `CMakeLists.txt` 中的同名变量为真时才会在生成的头文件中定义，区别于 `#define` 无论何时都会定义。

例如:

```shell
-> cat config.h.cmake

#ifndef __CONFIG_H
#define __CONFIG_H

/* Define to 1 if you have the <mach/mach_time.h> header file. */
#cmakedefine HAVE_MACH_MACH_TIME_H 1

#endif /* __CONFIG_H */
```

```shell
-> cat CMakeLists.txt

include(CheckIncludeFiles)
check_include_files("mach/mach_time.h" HAVE_MACH_MACH_TIME_H)
configure_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/config.h
        NEWLINE_STYLE UNIX)
list(APPEND HEADERS ${CMAKE_CURRENT_BINARY_DIR}/config.h)
```

只有当 `mach/mach_time.h` 存在时，在 `config.h` 才会定义 `HAVE_MACH_MACH_TIME_H` 。