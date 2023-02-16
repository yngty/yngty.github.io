---
title: Disk Arbitration
date: 2023-02-16 13:38:07
tags:
- MacOS
- Disk
categories:
- MacOS
---

# 介绍

`Disk Arbitration framework` 是一个基于 `Core Foundation` 的低级框架。会在磁盘出现和消失时通知您的应用程序，并让您的应用程序影响该过程。借助 `Disk Arbitration`，我们可以：

- 检测何时出现新磁盘
- 阻止挂载
- 使用不同的标志或在不同的安装点上安装卷
- 卸载卷
- 观察卷的变化

<!--more-->

# 使用磁盘仲裁通知和批准回调

1. 通过调用创建会话对象 `DASessionCreate`。
2. 如果您想知道磁盘相关事件何时发生，请注册通知回调；如果您想积极参与仲裁过程，请注册批准回调。
3. 在运行循环或调度队列上安排会话对象（并在必要时启动运行循环或调度队列）。
4. 处理您的应用收到的任何回调。
5. 当应用程序不再需要接收回调时，取消调度会话对象并释放它。

## 创建会话

编写磁盘仲裁通知客户端时必须做的第一件事是创建一个会话 (`DASessionRef`)。要创建磁盘仲裁会话，请调用`DASessionCreate`，如下所示：

```c
DASessionRef session;
session = DASessionCreate(kCFAllocatorDefault);
```

## 注册通知和批准

磁盘仲裁支持两种类型的回调。通知回调告诉您发生了某些事情。批准回调允许您阻止挂载、卸载或弹出操作发生。

### 通知回调

- `DADiskAppearedCallback`—出现磁盘或出现分区时调用
- `DADiskDescriptionChangedCallback`—当磁盘的描述发生变化时调用（在 `OS X v10.7` 及更高版本中，当首次安装卷时）
- `DADiskDisappearedCallback`—弹出可移动磁盘时调用
- `DADiskPeekCallback`—在首次探测磁盘时、自动挂载开始之前以及发送任何其他通知之前调用

### 注册函数

- `DARegisterDiskAppearedCallback`
- `DARegisterDiskDescriptionChangedCallback`
- `DARegisterDiskDisappearedCallback`
- `DARegisterDiskPeekCallback`

这些注册函数中的大多数都采用匹配字典。您通常应该传递 `NULL``（以匹配所有磁盘）或传递标准匹配字典，例如kDADiskDescriptionMatchMediaUnformatted`. 这些匹配字典的详细匹配行为如下所示:


| 标准字典 | 内容 | 描述 |
| :---        |    :---  |          :--- |
| kDADiskDescriptionMatchMediaUnformatted | kDADiskDescriptionMediaSizeKey价值为0 | 匹配未格式化的媒体（如空白 DVD）。|
| kDADiskDescriptionMatchMediaWhole | kDADiskDescriptionMediaWholeKey有价值true | 仅匹配整盘媒体（/dev/disk0例如 ）。|
| kDADiskDescriptionMatchVolumeMountable | kDADiskDescriptionVolumeMountableKey有价值true | 匹配可安装的卷。|
| kDADiskDescriptionMatchVolumeUnrecognized | kDADiskDescriptionVolumeMountableKey有价值false | 匹配不可挂载的磁盘。|

例如，要限制与 USB 连接媒体的匹配，您可以创建一个匹配字典，如下所示：

```c
CFMutableDictionaryRef matchingDict =
    CFDictionaryCreateMutable(
        kCFAllocatorDefault,
        0,
        &kCFTypeDictionaryKeyCallBacks,
        &kCFTypeDictionaryValueCallBacks);
 
CFDictionaryAddValue(matchingDict,
    kDADiskDescriptionDeviceProtocolKey,
    CFSTR(kIOPropertyPhysicalInterconnectTypeUSB));
```
[IOStorageProtocolCharacteristics.h User-Space Reference](https://developer.apple.com/documentation/kernel/kiopropertyphysicalinterconnecttypeusb)中描述了其他互连类型和其他相关常量。最后，只要磁盘事件与指定的匹配字典（或多个字典）和使用上下文指针的事件类型匹配，您就可以将任意数据传递给回调。通过传递不同的上下文指针，您可以使用不同的匹配字典多次注册相同的回调，并向回调提供指示哪个注册匹配的信息。如果您不需要提供此类上下文信息，只需传递NULL此参数即可。

每个回调的详细信息在以下部分中有更详细的描述。

### 注销通回调

当您不再需要通知回调时，通过调用取消注册 `DAUnregisterCallback`。例如：

```c
DAUnregisterCallback(session, mycallbackfuntion, NULL);
```
**请务必传入注册函数时使用的原始上下文指针值**。


### 批准回调

通过三种方式在磁盘仲裁中注册批准回调，具体取决于您希望何时收到通知。

- 如果您希望在弹出磁盘之前获得许可，请调用 `DARegisterDiskEjectApprovalCallback`.
- 如果您希望在安装卷之前获得许可，请调用 `DARegisterDiskMountApprovalCallback`.
- 如果您希望在卸载卷之前获得许可，请调用 `DARegisterDiskUnmountApprovalCallback`。

```c
DADissenterRef allow_mount(DADiskRef disk, void *context);
 
...
 
session = DASessionCreate(kCFAllocatorDefault);
 
DARegisterDiskMountApprovalCallback(session,
                NULL, /* Match all disks */
                allow_mount,
                NULL); /* No context */
 
...
 
DADissenterRef allow_mount(
        DADiskRef disk,
        void *context)
{
        int allow = 0;
 
        if (allow) {
                /* Return NULL to allow */
                fprintf(stderr, "allow_mount: allowing mount.\n");
                return NULL;
        } else {
                /* Return a dissenter to deny */
                fprintf(stderr, "allow_mount: refusing mount.\n");
                return DADissenterCreate(
                        kCFAllocatorDefault, kDAReturnExclusiveAccess,
                        CFSTR("It's mine!"));
        }
}


```
### 注销批准回调

当您不再需要批准回调时，您应该通过调用取消注册 `DAUnregisterApprovalCallback`。例如：
```c
DAUnregisterApprovalCallback(session, mycallbackfuntion, NULL);
```
**请务必传入注册函数时使用的原始上下文指针值**。

## 使用调度队列

```c
/* Schedule the session on a dispatch queue. */
DASessionSetDispatchQueue(session, queue);
 
/* Unschedule the session on a dispatch queue. */
DASessionSetDispatchQueue(session, NULL);
 
/* Clean up the session resources. */
CFRelease(session);
```
## 使用运行循环

```c
/* Schedule a disk arbitration session. */
DASessionScheduleWithRunLoop(session, CFRunLoopGetCurrent(), kCFRunLoopDefaultMode);
 
/* Start the run loop.  (Don't do this if you already have
   a running Core Foundation or Cocoa run loop.) */
CFRunLoopRun();
 
/* Clean up a session. */
DASessionUnscheduleFromRunLoop(session,
    CFRunLoopGetCurrent(), kCFRunLoopDefaultMode);
CFRelease(session);
```

# 操作磁盘和卷
## 获取磁盘对象
在您可以操作磁盘或卷之前，您必须`DADiskRef`为该磁盘或卷获取一个对象。通过四种方式获取对象：

- 作为传递给事件回调的参数（在使用磁盘仲裁通知和批准回调中描述）
- 通过调用从 `io_service_t`用户空间引用到有效设备切片的对象 `IOMediaDADiskCreateFromIOMedia`
- 您可以通过调用或io_service_t来获取对象的用户空间引用。
- 从 `BSD` 设备名称（disk1s1例如）使用 `DADiskCreateFromBSDName`
- 从挂载点调用 `DADiskCreateFromVolumePath`

如果您有一个`io_service_t`对象或一个 `BSD` 设备名称，您的应用程序可以创建一个`DADiskRef`对象，如下所示：

`DASessionRef`按照创建会话中的描述创建一个对象。
- 按照`Scheduling the Session with the Run Loop or Dispatch Queue`中的描述安排它。确保您的调度队列或运行循环正在运行。
- 创建磁盘对象。
- 根据需要操纵它们。

## 获取磁盘信息

盘仲裁提供三个函数来获取有关磁盘和分区的附加信息：`DADiskCopyDescription` 、`DADiskGetBSDName` 和`DADiskCopyIOMedia`。通常，您可以通过调用 获得关于特定磁盘的几乎所有信息 `DADiskCopyDescription`。然而，对于一些相当深奥的信息，您可能必须`IOMedia`为磁盘获取一个对象并查询该对象。

如果您需要磁盘或分区的 `BSD` 设备名称（`disk1s1`例如）作为 `C` 字符串（通常在使用 `POSIX` 级 `API` 时使用），请调用 `DADiskGetBSDName`.
对于大多数其他信息，请调用`DADiskCopyDescription`，如获取描述字典中所述。
如果无法通过 获得您需要的信息 `DADiskCopyDescription`，请调用DADiskCopyIOMedia。

### 获取描述字典

`DADiskCopyDescription`方法返回一个 `CFDictionaryRef` 对象，其中包含有关磁盘或分区的几十条信息。一些常用的数据包括：
- 挂载点和卷名
- `BSD` 设备节点名称和主要和次要编号
- 有关硬件的信息（`设备 ID` 、`供应商 ID` 、`GUID` 等）
- 连接信息（总线名称和路径

您可以在磁盘仲裁框架的标头中找到完整的属性列表 [DADisk.h Reference.](https://developer.apple.com/documentation/diskarbitration/dadisk_h?language=objc)，以及每个键值的预期数据类型的描述。

### 从 `I/O Kit` 获取附加信息

在极少数情况下，您可能需要获取有关磁盘的其他信息，而不是磁盘仲裁提供的信息。如果这样做，您可以调用`DADiskCopyIOMedia`以获取一个 `io_service_t`对象，该对象是对象的用户空间表示 `IOMedia`。您可以像操作任何 `I/O Registry` 对象一样操作此对象。

例如，您可以通过调用 `IORegistryEntryCreateCFProperties`结果对象来获取具有媒体 `I/O` 注册表属性的 `Core Foundation` 字典。

`I/O Registry` 字典中的属性在 `I/O Kit Framework` 中定义。有关详细信息，请参阅[I/O Kit Framework Reference](https://developer.apple.com/documentation/iokit)。


## 安装和卸载卷
- `DADiskMount`
- `DADiskMountWithArguments`

    ```c
    unsigned char *mppath = "/mnt/mydisk";
    
    path = CFURLCreateFromFileSystemRepresentation(
        kCFAllocatorDefault,
        mppath,
        strlen(mppath),
        true);
    
    DADiskMountWithArguments(disk, path, kDADiskMountOptionDefault,
        mount_complete_callback, NULL,
        NULL);
    ```
- `DADiskUnmount`
    ```c
    void unmount_done(DADiskRef disk,
    DADissenterRef dissenter,
    void *context);
 
    ...
    
    DADiskUnmount(disk, kDADiskUnmountOptionDefault,
        unmount_done, NULL);
    
    ...
    
    void unmount_done(DADiskRef disk,
        DADissenterRef dissenter,
        void *context)
    {
        if (dissenter) {
            /* Unmount failed. */
            char buf[MAXPATHLEN];
            if (CFURLGetFileSystemRepresentation(fspath, false, (UInt8 *)buf, sizeof(buf))) {
                fprintf(stderr, "Unmount failed (Error: 0x%x Reason: %s).  Retrying.\n",
                    DADissenterGetStatus(dissenter),
                    buf);
            } else {
                /* Something is *really* wrong. */
            }
        } else {
            /* Do something. */
        }
    }
    ```
## 弹出磁盘
在弹出磁盘之前，您必须卸载磁盘上的所有卷。首先调用 `DADiskUnmount`，将整个磁盘分区作为磁盘参数传递，并`kDADiskUnmountOptionWhole` 在卸载选项中设置标志。然后调用 `DADiskEject`
```c
void unmount_done(DADiskRef disk,
    DADissenterRef dissenter,
    void *context);
void eject_done(DADiskRef disk,
    DADissenterRef dissenter,
    void *context);
 
...
 
/* Unmount all volumes */
DADiskRef wholedisk = DADiskCopyWholeDisk(partition);
DADiskUnmount(wholedisk, kDADiskUnmountOptionWhole,
    unmount_done, NULL);
CFRelease(wholedisk);
 
...
 
/* In the unmount callback, eject the volume. */
void unmount_done(DADiskRef disk,
    DADissenterRef dissenter,
    void *context)
{
    if (dissenter) {
        ...
    } else {
        DADiskEject(disk, kDADiskEjectOptionDefault,
            eject_done, NULL);
    }
}
 
/* Eject callback. */
void eject_done(DADiskRef disk,
    DADissenterRef dissenter,
    void *context)
{
    if (dissenter) {
        ...
    } else {
        ...
    }
}
```
# 参考资料

* [Disk Arbitration Programming Guide](https://developer.apple.com/library/archive/documentation/DriversKernelHardware/Conceptual/DiskArbitrationProgGuide/)