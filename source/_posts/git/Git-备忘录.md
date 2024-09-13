---
title: Git 备忘录
date: 2024-09-13 10:38:55
tags:
- Git
categories:
- Git
---

# 如何修改 `Git`提交历史中的 `author` 等信息

## 修改上次提交的commit信息
```
git commit --amend --author="newname<newmail>"
// 不想修改提交信息，则添加--no-edit
git commit --amend --author="newname<newmail>" --no-edit
```
