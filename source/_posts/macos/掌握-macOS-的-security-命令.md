---
title: 掌握 macOS 的 security 命令
date: 2025-05-27 15:30:21
tags:
- Keychain
categories:
- MacOS
---

## 什么是 security？

在 `macOS` 系统中，`security` 命令是一个强大的工具，可以让你管理密钥链（`Keychain`）中的敏感信息，如密码、证书、私钥等。

---

<!--more-->

## 钥匙串管理

### 1. 基础操作
```bash
# 列出所有钥匙串
security list-keychains

# 创建新钥匙串（密码为 123）
security create-keychain -p 123 MyKeychain.keychain

# 解锁钥匙串
security unlock-keychain -p 123 MyKeychain.keychain
```

### 2. 设置默认钥匙串
```bash
security default-keychain -s ~/Library/Keychains/login.keychain
```
## 证书管理
### 1. 查找证书
```bash
# 列出所有代码签名证书
security find-identity -v -p codesigning
```
- **find-identity**：查找钥匙串中的身份标识（证书+私钥对）
- **-v**：详细模式，显示更多信息
- **-p codesigning**：只查找支持代码签名策略的证书

输出示例：
```bash
1) 1234567890ABCDEF1234567890ABCDEF12345678 "Apple Development: your@email.com (TEAMID)"
2) ABCDEF1234567890ABCDEF1234567890ABCDEF12 "Developer ID Application: Company Name (TEAMID)"
   2 valid identities found
```
# 查看证书详细信息
```bash
security find-certificate -c "Apple Development" -a -Z
```
- `-c "Apple Development"`：按证书名称过滤（支持模糊匹配）
- `-a`：搜索所有钥匙串（包括系统和登录钥匙串）
- `-Z`：显示证书的 `SHA-1` 哈希值


### 2. 导入/导出证书
```bash
# 导入 .p12 证书到登录钥匙串
security import cert.p12 -k ~/Library/Keychains/login.keychain -T /usr/bin/codesign
```
- import：导入操作

- cert.p12：PKCS#12 格式的证书文件

- -k：指定目标钥匙串路径

- -T /usr/bin/codesign：授权 codesign 工具访问此证书

### 导出证书为 PEM 格式
```bash
security export -k login.keychain -t certs -f pemseq -o certs.pem
```
- -t certs：导出类型为证书（非私钥）

- -f pemseq：输出格式为 PEM 序列（包含完整证书链）

- -o certs.pem：输出文件名

**提示**：`-T /usr/bin/codesign` 表示允许 `codesign` 访问此证书。

### 高级导出选项：
```bash
# 导出 PKCS#12 格式（包含私钥）
security export -k login.keychain -t identities \
  -f pkcs12 -P "strongPassword" -o full.p12

# 仅导出叶子证书
security export -k login.keychain -c "Developer ID" \
  -f pem -o single-cert.pem
```

## 密码管理
### 1. 存储密码
```bash
security add-generic-password \
  -a "admin" \
  -s "MySQL_Prod" \
  -w "s3cret" \
  -T /usr/bin/codesign
```
### 2. 读取密码（用于脚本自动化）
```bash
security find-generic-password -s "MySQL_Prod" -w
```

## 高级场景
### 1. 修复钥匙串权限
```bash
security set-key-partition-list -S "apple-tool:,apple:" -s -k "password" login.keychain
```
- 解决 `codesign` 因权限不足报错的问题。

### 2. 解析 iOS 描述文件
```bash
security cms -D -i embedded.mobileprovision
```

## 实战示例
### 场景 1：CI/CD 中的自动化解锁
```bash
# 解锁钥匙串（假设密码存储在环境变量中）
security unlock-keychain -p "$KEYCHAIN_PASSWORD" login.keychain

# 执行代码签名
codesign -s "Developer ID" App.app
```
### 场景 2：调试证书链
```bash
# 验证证书是否受信任
security verify-cert -c dev_cert.pem -k login.keychain

# 转换为可读格式
security find-certificate -c "Apple Development" -p | openssl x509 -text
```

## 参考资源
- [`Apple Keychain` 官方文档](https://developer.apple.com/documentation/security/keychain-services?language=objc)
- 终端手册：`man security`
