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

# cherry-pick 冲突

在 `Git` 中，若想选择性保留冲突文件的原始版本（当前分支的版本）或合并来的版本（被 `cherry-pick` 的提交版本），可以通过以下命令实现：

## 1. 保留当前分支的原始版本

```bash
# 针对特定冲突文件（例如你的 xxx.cpp）
git checkout --ours src/xxx.cpp

# 标记冲突已解决
git add src/xxx.cpp

# 继续完成 cherry-pick 流程
git cherry-pick --continue
```
## 2. 保留合并来的版本（冲突文件以被 cherry-pick 的提交内容为准）

```bash
# 针对特定冲突文件
git checkout --theirs src/xxx.cpp

# 标记冲突已解决
git add src/xxx.cpp

# 继续完成 cherry-pick 流程
git cherry-pick --continue
```

## 关键概念说明：

- **--ours**：代表当前分支的版本（即你在执行 `cherry-pick` 时的本地原始状态）。

- **--theirs**：代表被 `cherry-pick` 的提交的版本（即你要合并进来的修改）。

## 补充说明：
如果多个文件冲突，可以批量操作：

```bash
# 保留所有文件的当前分支版本
git checkout --ours .

# 或保留所有文件的合并来的版本
git checkout --theirs .
```
（操作后仍需 `git add` 并 `git cherry-pick --continue`）

如果中途想彻底放弃整个 `cherry-pick`，仍可用：

```bash
git cherry-pick --abort
```
