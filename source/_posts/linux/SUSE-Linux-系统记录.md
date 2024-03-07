---
title: SUSE Linux 系统记录
date: 2024-03-07 11:18:01
tags:
---

最近接触到 `SUSE Linux` 操作系统，一些命令不一样，这里记录下:

# 包管理器 `zypper`

- 包的安装

```shell
sudo zypper install 包名
```

- 搜索包

```shell
sudo zypper search 包名
```
# 老旧版本系统的包查找

我用的系统是 `SUSE 12 SP5` 基本没有可用的在线 `Repositories`, 一些包很难找到了，可以通过[SUSE Linux Enterprise Software Development Kit](https://www.suse.com/download/sle-sdk/) 下载对应的 `SDK ISO` 安装里面的 `rpm`。

# 参考资料

* [SUSE 12 SP3 的源管理相关](https://www.cnblogs.com/unchch/p/12910463.html)
