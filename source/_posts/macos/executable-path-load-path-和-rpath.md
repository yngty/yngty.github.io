---
title: "@executable path, @load path 和 @rpath"
date: 2023-08-25 11:05:29
tags:
- DYLD
categories:
- MacOS
---

在 `macOS` 上，动态链接器使用特定的路径变量来解析运行时的库位置。这些路径变量包括：绝对路径、 `@executable_path`、`@loader_path` 和 `@rpath`。

# 绝对路径

对于安装在系统中共享位置的框架很有用，一般是 `/Library/Frameworks/xxx`、 `/usr/lib/xxx`, 但是查找嵌入在应用内部的动态库就很难使用，应用安装的位置都不固定，所以引出新的方式。

# @executable path

`@executable_path` 是用于指代**当前正在执行的程序或应用的路径**。当你的应用程序或其动态库需要引用位于与可执行文件相同路径（或其子目录）下的其他动态库时，这会非常有用。
<!--more-->
举例说明: 
```
MyApp/
|-- MyApp.app/
    |-- Contents/
        |-- MacOS/
            |-- MyApp (可执行文件)
            |-- libA.dylib (动态库A)
        |-- Frameworks/
            |-- libB.dylib (动态库B)
```

假设 `MyApp` 依赖于 `libA.dylib`，而 `libA.dylib` 依赖于 `libB.dylib`。
如果你希望 `libA.dylib` 在运行时找到 `libB.dylib`，可以设置 `libB.dylib` 的安装名称为 `@executable_path/../Frameworks/libB.dylib`。当 `MyApp` 启动并加载 `libA.dylib` 时，`@executable_path` 将解析为 `MyApp.app/Contents/MacOS/`，因此 `libA.dylib` 会正确地找到 `libB.dylib` 在 `MyApp.app/Contents/Frameworks/` 目录下。

# @load path

**加载的动态库的路径**。在很多情况下，这个路径是基于正在加载该库的模块，因此它可能会随着加载该库的不同模块而变化。

举例说明:
```
MyApp/
|-- MyApp.app/
    |-- Contents/
        |-- MacOS/
            |-- MyApp (主应用程序)
            |-- libMain.dylib (主应用的动态库)
        |-- Plugins/
            |-- PluginA.bundle/
                |-- PluginA (插件)
                |-- libPluginA.dylib (插件的动态库)
```
在这个例子中，`MyApp` 依赖 `libMain.dylib`，而 `PluginA` 依赖 `libPluginA.dylib`。

- 使用 `@executable_path`：
 
  如果 `libMain.dylib` 需要引用与其位于同一目录下的另一个库，如 `libHelper.dylib`，它可以使用 `@executable_path/libHelper.dylib` 作为路径。但是，这对 `PluginA` 中的库不起作用，因为 `@executable_path` 总是指向 `MyApp.app/Contents/MacOS/`，不考虑加载它的实际模块。

- 使用 `@loader_path`：

  如果 `libPluginA.dylib` 需要引用与其位于同一目录下的另一个库，如 `libHelper.dylib`，它可以使用 `@loader_path/libHelper.dylib` 作为路径。这样，不管 `libPluginA.dylib` 被哪个模块加载，它都会正确地引用到相对于加载它的模块的库。

**如果是一个应用程序，那么 `@load path` 与 `@executable_path` 相同。**

# @rpath

`@rpath (Runtime Search Path)` 提供了一种动态方式来指定和查找动态库。它的主要优势是提供了更多的灵活性，尤其是在面对多种可能的库位置或多个版本的库时。

举例说明:
```
MyApp/
|-- MyApp.app/
    |-- Contents/
        |-- MacOS/
            |-- MyApp (可执行文件)
        |-- Frameworks/
            |-- libA.dylib (动态库A版本1)
        |-- Plugins/
            |-- PluginA/
                |-- libA.dylib (动态库A版本2)
```

假设 `MyApp` 可能需要加载两个版本中的任何一个 `libA.dylib`，具体取决于特定的运行时情境。使用` @rpath` 可以轻松管理这种情况。

- 设置 `@rpath`:
    
    在构建 `MyApp` 时，你可以设置多个运行时搜索路径（`rpaths`）:
    - `@executable_path/../Frameworks`
    - `@executable_path/../Plugins/PluginA`


- 使用 `@rpath` 在动态库中:

    设 `libA.dylib` 的安装名称为 `@rpath/libA.dylib`。

- 运行时解析:

    当 `MyApp` 需要加载 `libA.dylib` 时，它会沿着 `rpath` 列表搜索。首先在 `Frameworks` 文件夹中查找，然后在 `PluginA` 文件夹中查找。

这种方法的好处是，你可以轻松地将相同的库放在多个位置，而不需要为每个位置硬编码路径。此外，应用程序的用户或开发人员可以通过修改 `rpath` 来改变库的搜索顺序或位置。

# `install_name_tool`

要查看或修改一个可执行文件或动态库的 `rpath`，你可以使用 `otool` 和 `install_name_tool` 这两个命令行工具。

```shell
➜ otool -l WeChat

...
Load command 104
          cmd LC_LOAD_DYLIB
      cmdsize 64
         name /usr/lib/swift/libswiftsimd.dylib (offset 24)
   time stamp 2 Thu Jan  1 08:00:02 1970
      current version 9.0.0
compatibility version 1.0.0
Load command 105
          cmd LC_LOAD_DYLIB
      cmdsize 64
         name /usr/lib/swift/libswiftFoundation.dylib (offset 24)
   time stamp 2 Thu Jan  1 08:00:02 1970
      current version 1.0.0
compatibility version 1.0.0
Load command 106
          cmd LC_RPATH
      cmdsize 32
         path /usr/lib/swift (offset 12)
Load command 107
          cmd LC_RPATH
      cmdsize 48
         path @executable_path/../Frameworks (offset 12)
...

```

添加 `rpath`:
```
install_name_tool -add_rpath @executable_path/. a.out
```

修改 `rpath`:
```
install_name_tool -change libFoo @rpath/libFoo a.out
```