---
title: macOS 内核扩展编程初探
date: 2025-06-04 14:41:04
tags:
- KEXT
- 内核扩展
categories:
- MacOS
---

## 什么是 macOS 内核扩展（KEXT）？

`macOS` 内核扩展是一种动态链接库（通常后缀为 `.kext`），用于扩展 `XNU` 内核的功能。`KEXT` 可被用于编写驱动程序、系统安全模块、文件系统支持等底层功能。

在 `macOS Catalina（10.15）`之后，`Apple` 推出 `System Extensions` 和 `DriverKit`，逐步替代传统 `KEXT`，但 `KEXT` 仍广泛用于底层研究和安全相关开发。

<!--more-->

## 开发环境准备

要在 macOS 上开发内核扩展，你需要以下工具和配置：
- **内核签名的苹果开发者账号**
- **禁用 SIP（System Integrity Protection）**
    - 进入恢复模式 执行 `csrutil disable`
    - 在自己的实验机器上加载不签名禁用 `SIP` 也能运行
- **启用开发者 KEXT 加载权限**：
  - 进入恢复模式禁用安全策略

## 创建第一个 KEXT 工程

1. **打开 Xcode**
2. **选择 File > New > New Project。弹出新建工程面板**
3. **选择 macOS > Generic Kernel Extension。创建内核扩展工程**
3. **输入“HelloKext”作为内核扩展名称，输入Bundle ID，点击继续。**

### `HelloKext.cpp`

```cpp
#include <mach/mach_types.h>

kern_return_t HelloKext_start(kmod_info_t * ki, void *d);
kern_return_t HelloKext_stop(kmod_info_t *ki, void *d);

kern_return_t HelloKext_start(kmod_info_t * ki, void *d)
{
    printf("HelloKext loaded.\n");
    return KERN_SUCCESS;
}

kern_return_t HelloKext_stop(kmod_info_t *ki, void *d)
{
    printf("HelloKext unloaded.\n");
    return KERN_SUCCESS;
}
```
> 在编写内核扩展时，只能使用Kernel.framework中包含的头文件。应用其它Framework可能会导致扩展无法加载或执行失败
>
> 使用 IOLog 打印信息，可以通过 Console.app 或 log show 查看

## 编译内核扩展

1. 和编译普通 `Xcode` 工程一样，`Command+B` 快捷键编译就完成了。
2. 打开 `Terminal`，进入到新编译的内核扩展所在目录。
3. 执行命令 `kextlibs -xml HelloKext.kext`, 把返回的结果填入内核扩展的`Info.plist`中。
```
<key>OSBundleLibraries</key>
	<dict>
		<key>com.apple.kpi.iokit</key>
		<string>21.6</string>
	</dict>
```

## 加载内核扩展

1. 修改 `root:wheel` 用户
    ```
    sudo chown -R 0:0 HelloKext.kext
    ```
2. 加载前可以用命令 `kextutil  -print-diagnostics HelloKext.kext` 检查一遍扩展, 第一次加载 · 需要在系统便好设置中审批扩展启动，并重启
3. 执行命令 `log show --predicate 'senderImagePath contains "HelloKext"' --info` 查看日志
4. 执行命令 `sudo kextload HelloKext.kext`，加载内核，发现日志`HelloKext loaded.`打印了出来。
5. 执行命令 `kextstat | grep -v com.apple`, 可以看到自己的内核模块的确加载成功了。
6. 执行命令 `sudo kextunload HelloKext.kext` ，卸载内核，发现日志`HelloKext unloaded.`打印了出来。

> /Library/Extensions 目录下会开机自动加载

## 注意事项与后续替代

由于 `Apple` 加强了系统安全策略，从 `macOS 11 Big Sur` 起，传统 `KEXT` 开发受限。建议：

- 新驱动程序使用 `DriverKit`；
- 安全模块迁移到 `System Extensions`；
- 不推荐在主力机器上加载未签名 `KEXT`。

## 参考资料

- [Implementation of a Kernel Extension (KEXT)](https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/KernelProgramming/Extend/Extend.html#//apple_ref/doc/uid/TP30000905-CH220-TPXREF101)
