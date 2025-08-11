---
title: eBPF 入门
date: 2025-08-01 11:24:28
tags:
- eBPF
- Linux
- 性能分析
---

# 一、eBPF 是什么？
## 1. 简单理解
`eBPF` 就是一个**让你安全地在 `Linux` 内核中运行自定义代码的机制**。

它可以在内核中运行经过验证的“沙盒字节码”，而不需要修改内核源码，也不需要编写复杂的内核模块。

## 2. eBPF 的核心优势
- **安全**：内核会对 `eBPF` 程序进行严格验证（`Verifier`）

- **高性能**：字节码会被 `JIT` 编译成本地机器码

- **灵活**：几乎可以挂到内核的任何地方（网络、进程、文件系统等）

- **热插拔**：加载/卸载不影响系统运行

# 二、eBPF 运行架构

下面这张图，可以帮你快速理解 eBPF 的工作原理：
```sql
+------------------------------+
| 用户态 (User Space)          |
|                              |
|  eBPF 程序 (C / bpftrace)    |
|  ↓ clang 编译                |
|  eBPF 字节码 (.o)            |
+--------------+---------------+
               |
               v
+--------------+---------------+
| 内核态 (Kernel Space)        |
|                              |
|  eBPF 验证器 (Verifier)      |
|    - 检查安全性              |
|    - 防止死循环              |
|    - 检查内存访问范围        |
|                              |
|  JIT 编译器 (JIT Compiler)   |
|    - 转成本地机器码执行      |
|                              |
|  挂载到 Hook 点              |
|    - kprobe/tracepoint       |
|    - XDP/tc                  |
|    - socket filter           |
+------------------------------+
```

# 三、eBPF 常用挂钩点

|类型	|说明|	场景|
| :- | :- | :- |
|**Socket Filter**|	套接字层数据包过滤	|tcpdump、网络分析|
|**kprobe/kretprobe**|	内核函数入口/返回点插桩	|系统调用跟踪|
|**tracepoint**|	预定义内核事件	|性能分析|
|**tc BPF**|	网络流量控制	|限速、QoS|
|**XDP**	|网络驱动最底层数据包处理|	DDoS 防护|


# 四、eBPF 实战：追踪 TCP 连接

我们做一个小实验：当进程建立 `TCP` 连接时，打印 `PID`、目标 `IP`、端口。

## 1. 内核态 eBPF 程序
```c
// tcpconnect.bpf.c
#include "vmlinux.h"
#include <bpf/bpf_helpers.h>
#include <bpf/bpf_endian.h>

char LICENSE[] SEC("license") = "GPL";

SEC("kprobe/tcp_v4_connect")
int tcp_connect(struct pt_regs *ctx)
{
    struct sock *sk = (struct sock *)PT_REGS_PARM1(ctx);
    u32 pid = bpf_get_current_pid_tgid() >> 32;

    __be32 daddr = 0;
    __be16 dport = 0;

    bpf_probe_read_kernel(&daddr, sizeof(daddr), &sk->__sk_common.skc_daddr);
    bpf_probe_read_kernel(&dport, sizeof(dport), &sk->__sk_common.skc_dport);

    bpf_printk("PID %d -> %pI4:%d\n", pid, &daddr, bpf_ntohs(dport));
    return 0;
}
```

## 2. 用户态加载器
```c
// loader.c
#include <stdio.h>
#include <bpf/libbpf.h>
#include <unistd.h>

static int libbpf_print_fn(enum libbpf_print_level level, const char *format, va_list args)
{
    return vfprintf(stderr, format, args);
}

int main()
{
    struct bpf_object *obj;
    struct bpf_program *prog;
    struct bpf_link *link;
    int err;

    libbpf_set_print(libbpf_print_fn);

    obj = bpf_object__open_file("tcpconnect.bpf.o", NULL);
    if (libbpf_get_error(obj)) {
        fprintf(stderr, "打开 eBPF 对象失败\n");
        return 1;
    }

    err = bpf_object__load(obj);
    if (err) {
        fprintf(stderr, "加载 eBPF 程序失败\n");
        return 1;
    }

    prog = bpf_object__find_program_by_name(obj, "tcp_connect");
    if (!prog) {
        fprintf(stderr, "找不到 eBPF 程序\n");
        return 1;
    }

    link = bpf_program__attach_kprobe(prog, false, "tcp_v4_connect");
    if (libbpf_get_error(link)) {
        fprintf(stderr, "附加 kprobe 失败\n");
        return 1;
    }

    printf("成功加载并附加 eBPF 程序。按 Ctrl+C 停止\n");
    while (1) {
        sleep(1);
    }

    bpf_link__destroy(link);
    bpf_object__close(obj);
    return 0;
}
```
## 3. 编译运行

```bash
# 编译内核态
clang -O2 -g -target bpf -D__TARGET_ARCH_x86 -c tcpconnect.bpf.c -o tcpconnect.bpf.o

# 编译用户态
gcc loader.c -o loader -lbpf

# 运行
sudo ./loader
```
## 4. 查看日志
```bash
sudo cat /sys/kernel/debug/tracing/trace_pipe
```

示例输出：

```
PID 1234 -> 93.184.216.34:80
PID 5678 -> 142.250.72.238:443
```

# 五、eBPF 的实际应用
1. 性能分析
    - bcc / bpftrace 脚本库

2. 网络安全

    - XDP 高速包过滤

3. 故障排查

    - 内核函数动态追踪

4. 容器可观测性

    - Cilium、Pixie

# 六、总结
eBPF 已经成为 Linux 内核可观测性和网络安全的核心技术。
它的强大在于：

- 不改内核源码

- 高性能

- 高安全

- 热插拔

对于开发者和运维工程师来说，掌握 `eBPF`，不仅可以提高故障排查效率，还能为安全和性能优化带来新的可能性。
