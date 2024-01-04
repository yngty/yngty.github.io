---
title: macOS Ventura App 管理
date: 2024-01-04 14:34:10
tags:
- macOS
categories:
- macOS
---

从 `macOS Monterey` 开始，如果应用程序被未由相同开发团队签名且未由 `NSUpdateSecurityPolicy` 允许的东西修改，`macOS` 将阻止修改并通知用户应用程序希望管理其他应用程序。点击通知会将用户发送到系统设置，他们可以在那里允许应用程序更新和修改其他应用程序。

![appmanager](/images/appmanager.png)

<!--more-->

- 从 `macOS Ventura` 开始，`Gatekeeper` 将检查所有经过公证的应用程序的完整性，而不仅仅是隔离的应用程序。
- 从 `macOS Ventura` 开始，如果您的认证应用程序的签名不再有效，`Gatekeeper` 将在**首次启动**时阻止其运行。
  - `macOS Monterey` 及更早版本仅在修改后的已认证应用程序仍**处于隔离状态**时才会阻止首次启动。
- 从 `macOS Ventura` 开始，`Gatekeeper` 还将阻止应用以某些方式修改您的应用程序。
- **完全磁盘访问自动包含应用程序管理权限**。即使在系统设置中禁用了应用程序管理权限。
- 应用程序如何更新
  - 由同一开发者帐户或团队有效签名的应用程序将继续能够互相更新。这将自动进行。
  - 要允许另一个开发团队更新您的应用程序或限制仅由您的更新程序更新，需要在 `info-plist`，添加 `NSUpdateSecurityPolicy` 。在 `NSUpdateSecurityPolicy` 内部，添加"`AllowProcesses`"，一个将团队标识符映射到签名标识符数组的字典。


# 参考资料

* [What’s new in privacy](https://developer.apple.com/videos/play/wwdc2022/10096/)
* [How macOS Ventura App Management works and doesn't work](https://lapcatsoftware.com/articles/AppManagement.html)
