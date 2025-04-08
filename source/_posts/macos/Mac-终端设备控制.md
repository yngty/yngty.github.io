---
title: Mac 终端设备控制
date: 2025-02-24 14:05:09
tags:
---

# AirDrop

- 打开
    ```shell
    launchctl load /System/Library/LaunchAgents/com.apple.sharingd.plist
    ```
- 关闭
    ```
    launchctl unload /System/Library/LaunchAgents/com.apple.sharingd.plist
    ```
# Bluetooth

- 关闭
    ```
    defaults write /Library/Preferences/com.apple.Bluetooth.plist ControllerPowerState 0

    killall bluetoothd
    killall blued
    ```
# CD

- eject
    ```
    for (DRDevice *device in [DRDevice devices]) {
        [device ejectMedia];
    }
    ```

# USB

使用 `Disk Arbitration framework`

`Disk Arbitration framework` 是一个基于 `Core Foundation` 的低级框架。会在磁盘出现和消失时通知您的应用程序，并让您的应用程序影响该过程。借助 `Disk Arbitration`，我们可以：

- 检测何时出现新磁盘
- 阻止挂载
- 使用不同的标志或在不同的安装点上安装卷
- 卸载卷
- 观察卷的变化
