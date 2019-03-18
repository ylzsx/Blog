####  Android系统架构
1. Android系统大致分为四层：Linux内核层、库和运行时、Framework层、应用层。
- Linux层
	- 其中包含了Android系统的核心服务，包括硬件驱动、进程管理、安全系统等。

#### Dalvik和ART
- Dalvik包含了一套Android运行环境虚拟器，每个App都会分配Dalvik虚拟机来保证相互之间不受干扰，保持独立。其特点为运行时编译。
- ART模式在Android 5.X版本后取代了Dalvik，其特点是安装时进行编译。

#### Android系统上下文
- Android系统的上下文对象，即在Context中。
- Activity、Service、Application都继承自Context。
	- 当应用第一次启动时，会创建一个Application对象，同时创建Application Context。我们可通过getApplicationContext()方法获取到整个App的Context。
	- 创建Activity和Service组件时，系统也会提供运行时的上下文环境。