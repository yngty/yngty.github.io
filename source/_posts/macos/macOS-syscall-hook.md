---
title: macOS syscall hook
date: 2025-07-08 09:45:24
tags:
---

## 1. 找到sysent

### 1. find_kernel_base
        #define MACH_KERNEL_BASE 0xFFFFFE0007004000 // macOS 内核文件的基地址 for "/System/Library/Kernels/kernel.release.t8101"
        使用 vm_kernel_unslide_or_perm_external 找到 kernel_slide
        base_address = kernel_slide + MACH_KERNEL_BASE
