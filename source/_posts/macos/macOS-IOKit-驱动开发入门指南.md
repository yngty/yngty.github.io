---
title: macOS IOKit 驱动开发入门指南
date: 2025-07-04 17:36:58
tags:
- macOS
- IOKit
- 驱动开发
- 内核扩展
- Xcode
- kext
categories:
- MacOS
---

### 完整调用流程

```mermaid
sequenceDiagram
    participant UserSpace as 用户空间程序
    participant IOKit as IOKit框架
    participant Driver as 内核驱动
    participant UserClient as UserClient实例

    Note over UserSpace,UserClient: 阶段1：服务发现
    UserSpace->>IOKit: IOServiceGetMatchingServices(kIOMainPortDefault, matchingDict, &iterator)
    IOKit->>Driver: 内核遍历服务列表
    Driver-->>IOKit: 返回匹配的服务对象
    IOKit-->>UserSpace: 通过iterator返回服务句柄

    Note over UserSpace,UserClient: 阶段2：建立连接
    UserSpace->>IOKit: IOServiceOpen(service, mach_task_self(), type, &connection)
    IOKit->>Driver: 调用驱动的newUserClient()

    alt 自定义newUserClient实现
        Driver->>Driver: 1. 安全检查(task_is_privileged等)
        Driver->>Driver: 2. 通过OSTypeAlloc创建UserClient
        Driver->>UserClient: initWithTask(owningTask, type...)
        UserClient-->>Driver: 返回初始化结果
        Driver-->>IOKit: 返回UserClient实例
    else 默认实现
        IOKit->>UserClient: 自动创建标准UserClient
        UserClient->>UserClient: 默认initWithTask()
        UserClient-->>IOKit: 返回实例
    end

    IOKit-->>UserSpace: 返回connection句柄

    Note over UserSpace,UserClient: 阶段3：命令交互
    loop 多次调用
        UserSpace->>UserClient: IOConnectCallMethod(connection, selector, ...)
        UserClient->>UserClient: 根据sMethods分派到具体方法
        UserClient->>Driver: 调用驱动方法（如performCalculation）
        Driver-->>UserClient: 返回数据/状态
        UserClient-->>UserSpace: 返回结果
    end

    Note over UserSpace,UserClient: 阶段4：断开连接
    UserSpace->>UserClient: IOServiceClose(connection)
    UserClient->>UserClient: clientClose()
    UserClient->>Driver: 通知驱动(可选)
    UserClient->>UserClient: 释放资源
```

## IODataQueueMemory

### 用户层代码

```
void ControllerImpl::MsgRecvThreadFunc() {
    kern_return_t kr;
    UInt32 dataSize;
    IODataQueueMemory* queueMappedMemory;
    vm_size_t queueMappedMemorySize;
    mach_vm_address_t address = 0;
    mach_vm_size_t size = 0;
    mach_port_t recvPort;
    TestMessage msg;

    // allocate a Mach port to receive notifications from the IODataQueue
    if ((recvPort = IODataQueueAllocateNotificationPort()) == MACH_PORT_NULL) {
        return;
    }

    // this will call registerNotificationPort() inside our user client class
    kr = IOConnectSetNotificationPort(m_nConnection, QUEUETYPE_MONITOR, recvPort, 0);
    if (kr != kIOReturnSuccess) {
        mach_port_destroy(mach_task_self(), recvPort);
        return;
    }

    // this will call clientMemoryForType() inside our user client class
    kr = IOConnectMapMemory(m_nConnection, kIODefaultMemoryType, mach_task_self(), &address, &size, kIOMapAnywhere);
    if (kr != kIOReturnSuccess) {
        mach_port_destroy(mach_task_self(), recvPort);
        return;
    }

    queueMappedMemory = (IODataQueueMemory*)address;
    queueMappedMemorySize = size;

    while (!m_bStop && IODataQueueWaitForAvailableData(queueMappedMemory, recvPort) == kIOReturnSuccess) {
        while (!m_bStop && IODataQueueDataAvailable(queueMappedMemory)) {
            dataSize = sizeof(msg);
            kr = IODataQueueDequeue(queueMappedMemory, &msg, &dataSize);
            if (kr != kIOReturnSuccess) {
                continue;
            }

           Callback(msg);

        }  // end while (IODataQueueDataAvailable(queueMappedMemory))
    }      // end while (IODataQueueWaitForAvailableData

exit:

    kr = IOConnectUnmapMemory(m_nConnection, kIODefaultMemoryType, mach_task_self(), address);
    if (kr != kIOReturnSuccess) {
    }

    mach_port_destroy(mach_task_self(), recvPort);

}
```
