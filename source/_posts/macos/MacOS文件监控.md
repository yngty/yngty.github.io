---
title: MacOS文件监控
date: 2024-10-25 10:25:38
tags:
- macOS
categories:
- macOS
---

# 1. 文件系统事件：文件变动的探测器

文件系统事件是macOS提供的一个API，可以帮助我们监听文件或目录的变更。只要文件或目录发生变化，这个API就会发出通知。

# 2. 如何在macOS上实施文件监控？

在macOS上进行文件监控，你可以使用以下两种方法：

## 2.1 FSEvents API

FSEvents API是一个低级的C语言API，可以让你直接与文件系统事件驱动程序进行交互。这种方法非常高效，但它也比较复杂。

## 2.2 Cocoa NSFilePresenter API

Cocoa NSFilePresenter API是一个更高层次的API，它封装了FSEvents API，使用起来更加简单。但它可能不如FSEvents API那么高效。


```
/*
USB设备类型：
DAVolumeKind = msdos、exfat、hfs;
msdos：MS-DOS
exfat：EXFAT
hfs：Mac OS 日志式、加密式


共享文件类型：
DAVolumeKind = smbfs;

*/
```
