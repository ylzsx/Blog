# 预备知识

## Android系统架构

Android操作系统采用五层的系统架构，从下到上分别如下：

- 内核层：该层包含了Linux内核以及Android在Linux内核上增加的驱动。这些驱动通常与硬件无关，而是为了上层软件服务的。
  - **Binder**：进程间通信基础设施。Binder在Android系统中大量使用，几乎所有的Framework层的服务都是通过Binder的形式暴露出接口供外部使用。
  - **Ashmem匿名共享内存**。共享内存的作用是当多个进程需要访问同一块数据时，可以避免数据复制。【例如：经由ContentProvider接口获取数据的客户端和ContentProvider之间就是通过共享内存的方式来访问的。】
  - **lowmemorykiller进程回收模块**。在Framework层，所有的应用进程都是由ActivityManagerService来管理的，它会根据进程的重要性设置一个优先级，这个优先级会被lowmemorykiller读取。在系统内存较低时，lowmemorykiller会根据优先级顺序，将优先级低的进程杀死，知道系统恢复到合适的内存状态。
  - **logger日志相关**。开发人员经常会使用logcat读取日志来帮助分析问题。而无论是logcat工具，还是通过日志API写入日志，最终都是由底层的logger驱动进行处理的。
  - **wakelock电源管理相关**。Android系统通常运行在以电池供电的移动设备上，因此专门增加了该模块来管理电源。
  - **Alarm闹钟相关**。为AlarmManager服务。
- 硬件抽象层：该层为硬件厂商定义了一套标准的接口。有了这套标准接口之后，可以在不影响上层的情况下，调整内部结构。
- Runtime和公共库层：该层包括了虚拟机和基本的运行环境。【早期的Android虚拟机是Dalvik，Android5.0上切换为Android RunTime。】
- Framework层：该层包含了一系列重要的系统服务。对于App层的管理和App使用的API基本上都是在这一层上提供的。下面介绍几个常见服务。
  - ActivityManagerService：负责Android所有应用组件的管理，以及App进程管理。
  - WindowManagerService：负责窗口管理。
  - PackageManagerService：负责APK包的管理，包括安装、卸载、更新等。
  - NotificationManagerService：负责通知管理。
  - PowerManagerService：电源管理。
  - LocationManagerService：定位相关的管理。
- 应用层：该层与用户直接接触。

## AOSP（Android Open Source Project）

window10安装方式：https://blog.csdn.net/weixin_41620505/article/details/88680355

## Binder机制

