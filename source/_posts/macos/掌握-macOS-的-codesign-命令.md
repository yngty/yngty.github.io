---
title: 掌握 macOS 的 codesign 命令
date: 2025-05-27 15:19:30
tags:
- 代码签名
categories:
- MacOS
---

## 什么是 codesign？

`codesign` 是 `macOS` 和 `iOS` 开发中用于**代码签名**的核心命令行工具，它确保应用程序的来源可信且未被篡改。无论是发布到 `App Store` 还是独立分发，代码签名都是必经流程。

---

<!--more-->

## 核心功能

1. **身份验证**
   - 通过 `Apple` 开发者证书验证开发者身份。
2. **完整性校验**
   - 确保应用在签名后未被非法修改。
3. **沙盒权限控制**
   - 通过 `entitlements` 文件定义应用权限（如网络访问、摄像头调用）。
4. **系统兼容性**
   - `macOS` 和 `iOS` 应用必须签名才能运行或分发。

---

## 基础操作

### 1. 对应用签名
```bash
codesign -s "Developer ID Application: Your Name (TeamID)" /path/to/App.app
```
- `-s` 指定证书名称（需与钥匙串中的名称完全匹配）。
- 查看可用证书：
    ```bash
    security find-identity -v -p codesigning
    ```
### 2. 验证签名

```bash
# 查看签名详细信息
codesign -dv /path/to/App.app

# 严格验证签名有效性
codesign -vvv /path/to/App.app
```
### 3. 提取权限配置（Entitlements）
```bash
codesign -d --entitlements - /path/to/App.app
```
## 高级用法
### 1. 强制重签名

```bash
codesign -f -s "New-Certificate" /path/to/App.app
```
- `-f` 参数强制覆盖现有签名。

**注意：重签名前建议先移除旧签名**：

```bash
codesign --remove-signature /path/to/App.app
```

### 2. 嵌套组件签名
```bash
codesign -s "Certificate" --deep /path/to/App.app
```
- `--deep` 会自动签名 `.app` 内的 `.framework` 和 `.dylib`，但对复杂项目可能失效。

### 3. 启用强化运行时（`Hardened Runtime`）
```bash
codesign -s "Certificate" \
  --options runtime \
  --entitlements entitlements.plist \
  /path/to/App.app
```
- 必须提供 `entitlements.plist` 文件，否则应用可能崩溃。

## 实战场景
### 场景 1：重签名整个应用
```bash
# 1. 移除旧签名
codesign --remove-signature App.app

# 2. 签名嵌套组件
find App.app -name "*.framework" -o -name "*.dylib" | xargs -I {} codesign -s "Certificate" {}

# 3. 签名主程序
codesign -s "Certificate" --entitlements entitlements.plist App.app
```
### 场景 2：解决常见错误
- **错误**：`"code object is not signed at all"`

    **原因**：嵌套代码未签名。

    **解决**：手动签名所有组件或使用 `--deep`。

- **错误**：`"resource modified"`

    **原因**：签名后文件被修改。

    **解决**：清理构建目录并重新签名。

## 调试工具
### 1. 查看完整签名信息
```bash
codesign -dvvv /path/to/App.app
```
### 2. 验证 Gatekeeper 兼容性
```bash
spctl -a -vvv /path/to/App.app
```
## 参考资源

- `Apple` 官方代码签名指南
- 终端手册：`man codesign`
