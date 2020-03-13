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

官方网址：https://source.android.com

中国地区网址：https://source.android.google.cn

获取可以编译AOSP源码的环境：https://source.android.google.cn/setup/initializing

获取AOSP代码：https://source.android.google.cn/setup/downloading

window10安装方式：https://blog.csdn.net/weixin_41620505/article/details/88680355

以下代码取自Android 7.1，所使用的代码Tag是android-7.1.1_rl。

## Binder机制

1. Android Binder是在OpenBinder框架上的定制实现，Binder是Android系统中的IPC(Inter-Process Communication，进程间通信)机制，无论是应用程序对系统服务的请求，还是应用程序自身提供对外服务，都会使用Binder。

2. 在UNIX/Linux环境下，传统的IPC机制包括：管道、消息队列、共享内存、信号量、Socket等。但Android系统中对于传统的IPC使用较少，大部分场景下使用的IPC均为Binder。
3. Binder相较于传统IPC更适合于Android系统的原因：
   - Binder本身是C/S架构，更适合Android系统架构。
   - 相较于传统IPC（管道、消息队列、Socket通信）都要两次数据复制，而Binder只需要一次，性能更优。
   - 安全性更好：传统IPC形式无法得到对方身份标识（UID/GID），而在使用Binder IPC时，该身份标识会跟随调用过程自动传递。Server端可以很容易知道Client端的身份，便于安全检查。

### Binder整体架构

- Binder实现分为以下几层：

  - Framework层：Java、JNI、C++部分	[JNI：Java Native Interface，提供了若干的API实现Java和其他语言(C/C++)等的通信]
  - 驱动层：位于Linux内核中，提供了最底层的数据传递、对象标识、线程管理、调用过程控制等功能，**驱动层是整个Binder机制的核心**。

- Binder框架是C/S框架，Client对于Server的请求会经由Binder框架由上至下传递到内核的Binder驱动中，请求中包含了Client将要调用的命令和参数。请求到了Binder驱动，在提供了服务的提供方之后，会再从下到上将请求传递给具体的服务。

  ![Binder调用机制](/resource/Binder调用过程.png)

- **ServiceManager**

  - 在同一时刻，系统中会有大量的Server同时存在（系统服务、三方应用都会使用）,因此为了确定Client在请求Server时，请求发送给哪个Server的问题，每个Server都需要一个唯一的标识id(其实就是一个字符串)，同时需要一个组织统一管理该标识，即为ServiceManager。

  - 每个公开对外提供服务的Server都需要通过`addService`注册到ServiceManager中，同时指定唯一标识id，Client要对Server发请求时，需先根据Server的id通过ServiceManager拿到Server的标识（通过`getService`），然后通过该标识与Server进行通信。

  - 实际中，为了便于使用，ServiceManager本身也实现了一个Server对象。Binder机制为ServiceManager预留了一个特殊的位置，当有进程通过`ioctl`并指定命令为`BINDER_SET_CONTEXT_MGR`时，驱动会认定该进程为ServiceManager。

  - ServiceManager先于所有Server之前启动，在它启动完成并告知Binder驱动之后，驱动便设定好了这个特定的节点。

    ```c
    // binder.c
    static struct binder_node *binder_context_mgr_node;
    ```

    > 注：每一个Binder Server在驱动中都会有一个`binder_node`与之对应。

### 驱动层

源码路径（该部分的源码不在AOSP中，而是位于Linux内核代码中）。

```
/kernel/divers/android/binder.c
/kernel/include/uapi/linux/android/binder.h
```

Binder是一个miscdev类型的驱动（混杂驱动）,其本身不对应任何硬件，所有的操作都在软件层。

`binder_init`函数负责Binder驱动的初始化工作，该函数大部分代码通过`debugfs_create_dir`和`debugfs_create_file`函数创建`debugfs`对应的文件。`binder_init`函数中最主要的工作如下：

```c
static struct miscdevice binder_miscdev = {
    .minor = MISC_DYNAMIC_MINOR,
    .name = "binder",
    .fops = &binder_fops
}
...
ret = misc_register(&binder_miscdev);
```

这里指定了Binder设备的名称是“binder”。这样，在用户空间便可以通过对`/dev/binder`文件进行操作使用Binder。`binder_miscdev`同时也制定了该设备的fops。fops也是一个结构体，该结构体中除`owner`外，每一个字段都是一个函数指针，这些函数指针对应了用户空间在使用Binder设备时的操作。比如，需要使用Binder的进程，几乎都是先通过`binder_open`打开Binder设备，后通过`binder_mmap`进行内存映射，之后通过`binder_ioctl`来进行实际的操作。Client对于Server的请求，以及Server对于Client请求结果的返回，都是通过`ioctl`完成。

```c
// binder.c
static const struct file_operations binder_fops = {
    .owner = THIS_MODULE,
    .poll = binder_poll,
    .unlocked_ioctl = binder_ioctl,
    .compat_ioctl = binder_ioctl,
    .mmap = binder_mmap,
    .open = binder_open,
    .flush = binder_flush,
    .release = binder_release,
}
```

#### 主要结构

Binder驱动中包含了很多的结构体。一类是用来与用户共用的，这些结构体在Binder通信协议过程中会用到。

| 结构体名称              | 说明                   |
| ----------------------- | ---------------------- |
| binder_write_read       | 存储一次读写操作的数据 |
| binder_transaction_data | 储存一次事务的数据     |

另一类仅仅Binder驱动内部实现过程中需要，他们定义在`binder.c`中。
| 结构体名称    | 说明                                   |
| ------------- | -------------------------------------- |
| binder_node   | 描述Binder实体节点，即对应了一个Server |
| binder_ref    | 描述对于Binder实体的引用               |
| binder_buffer | 描述Binder通信过程中储存数据的Buffer   |
| binder_proc   | 描述使用Binder的进程                   |
| binder_thread | 描述使用Binder的线程                   |

#### Binder协议

Binder协议可以分为控制协议和驱动协议两类：

- 控制协议是进程通过`ioctl("/dev/binder")`与Binder设备进行通信的协议，该协议包含的重要命令：

  | 命令              | 说明 | 参数类型 |
  | ----------------- | -------------------------------- | ---- |
  | BINDER_WRITE_READ | 读写操作，最常用的命令。IPC过程就是通过这个命令进行数据传递 | binder_write_read |

- 驱动协议描述了Binder驱动的具体使用过程。驱动协议又分为两类：

  - `binder_driver_command_protocol`，描述了进程发送给Binder驱动的命令。
  - `binder_driver_return_protocol`，描述了Binder驱动发送给进程的命令。

我们以一次Binder请求过程来详细理解一下Binder协议的通信：

![Binder协议](/resource/Binder协议.png)

- 驱动协议命令（进程发送给驱动）

  | 命令            | 说明                                 | 参数类型                |
  | --------------- | ------------------------------------ | ----------------------- |
  | BC_ENTER_LOOPER | 通知驱动主线程ready                  | void                    |
  | BC_TRANSACTION  | Binder事务，即Client对于Serve的请求  | binder_transaction_data |
  | BC_REPLY        | 事务的应答，即Server对于Client的回复 | binder_transaction_data |

- 返回类型（驱动发送给进程）
  | 返回类型                | 说明                                 | 参数类型                |
  | ----------------------- | ------------------------------------ | ----------------------- |
  | BR_TRANSACTION_COMPLETE | 驱动对于接受请求的确认回复           | void                    |
  | BR_TRANSACTION          | 通知进程收到一次Binder请求(Server端) | binder_transaction_data |
  | BR_REPLY                | 通知进程收到Binder请求的回复(Client) | binder_transaction_data |

Binder协议的通信过程中，不仅仅是发送请求和接收数据这些命令，同时也包括了对于引用计数的管理和对于死亡通知的管理（告知一方，通信的另一方已经死亡）等功能。

#### 打开Binder设备

- 任何进程在使用Binder前，都会通过用户空间的`open("/dev/binder")`调用驱动中的`binder_open`函数。在该函数中，会创建进程相应的`binder_proc`对象，并将其添加到列表`binder_procs`中。

  > 注：为了便于查找，驱动内部结构体之间都留有字段储存关联结构。
  >
  > ![Binder Driver中的主要数据结构](/resource/BinderDriver.png)

- 在打开Binder设备后，进程通过`mmap`进行内存映射，`mmap`作用如下：

  - 申请一块内存空间，用来接收Binder通信过程中的数据。
  - 对这块内存进行地址映射，以便将来访问

  `binder_mmap`函数中，会申请一块物理内存，将用户空间和内核空间同时对应到这块内存上。当Client要发送数据给Server时，**只需一次**，将Client发送过来的数据复制到Server端的内核空间指定的内存地址即可，由于这个内存地址在服务端已经同时映射到用户空间，因此无须再做一次复制，Server即可直接访问。

  ![Binder内存映射](/resource/Binder内存映射.png)

#### 内存管理

- 进程通过`ioctl`系统调用`binder_ioctl`函数发出请求，该函数第一个参数为打开Binder设备时的fd，第二个参数为具体的操作码，第三个参数为请求数据`bwr`（类型为`binder_write_read`）。当操作码为`BINDER_WRITE_READ`，且

  - 若`bwr.write_size>0`，则调用`binder_thread_write`；

  - 若`bwr.read_size>0`，则调用`binder_thread_read`。

    ![binder_write_read与binder_transaction_data`结构](/resource/binder_write_read结构.png)

- 当Client请求Server时，便发送一个`BINDER_WRITE_READ`命令，同时框架会将实际数据包装好，从而执行`binder_thread_write`方法，此时，`binder_transaction_data`中的`code`是`BC_TRANSACTION`驱动协议命令，从而会调用`binder_transaction`方法通知Binder驱动对一次Binder事务进行处理，其中会调用`binder_alloc_buf`函数为此次事务申请一个缓存，之后会通过`binder_update_page_range`函数进行内存分配并完成内存映射。

  ![binder_thread_write调用关系](/resource/binder_thread_write调用关系.png)

- 我们通过`BC_FREE_BUFFER`驱动命令经`binder_free_buf`函数实现内存的释放。在该函数中，所做的事情包括：

  - 更新计算进程的空闲缓存大小。
  - 通过`binder_update_page_range`释放内存。
  - 更新`binder_proc`的`buffers、free_buffers、allocated_buffers`字段。

#### 驱动传递对象

- Binder机制淡化了进程的边界，使得跨越进程也能够调用到指定服务的方法，其原因是Binder机制在底层处理了在进程间的“对象”传递。在Binder驱动中，通过特定的标识`flat_binder_object`来进行对象的传递。

  ```c
  // binder.h
  struct flat_binder_object {
      __u32 type;
      __u32 flags;
      
      union {
          binder_uintptr_t binder; // local object
          __u32 handle;	// remote object
      };
      binder_uintptr_t cookie;
  }
  
  // type 有如下5种类型
  enum {
      BINDER_TYPE_BINDER = B_PACK_CHARS('s', 'b', '*', B_TYPE_LARGE),
      BINDER_TYPE_WEAK_BINDER = B_PACK_CHARS('w', 'b', '*', B_TYPE_LARGE),
      BINDER_TYPE_HANDLE = B_PACK_CHARS('s', 'h', '*', B_TYPE_LARGE),
      BINDER_TYPE_WEAK_HANDLE = B_PACK_CHARS('w', 'h', '*', B_TYPE_LARGE),
      BINDER_TYPE_FD = B_PACK_CHARS('f', 'd', '*', B_TYPE_LARGE),
  }
  ```

- 当发送对象传递到Binder驱动时，驱动会对数据流中的`flat_binder_object`做相应的翻译和解释（**改变`type`类型，为Binder在接收过程中创建位于内核中的引用并将引用号填入`handle`中**），然后传递到接收的进程。由于每个请求和请求的返回都会经历驱动的翻译，故该过程从进程角度看是完全透明的。

#### <font color='red'>个人总结</font>

Binder执行流程：

- 初始化`binder_init`，向内核中注册Binder设备。
- `binder_open`打开Binder设备：创建`binder_proc`对象，并加入`binder_procs`中。
- `binder_mmap`进程内存映射。
- Client通过`binder_ioctl`发送请求，经Binder协议通知Server。

### Framework层

#### C++部分

Binder库最终会编译为一个动态链接库`libbinder.so`，供其他进程链接使用。下文中，我们将Binder Framework的C++部分称为`libbinder`。

在`libbinder`中，将实现分为`Proxy`和`Native`两端。`Proxy`(远程端)代表调用方，通常与服务的实现不在同一个进程，即`Client`端，是服务对外提供的接口；`Native`端(本地端)是服务实现的自身，即`Server`端。

![libbinder中的主要结构](/resource/libbinder中的主要结构.png)

##### 重要的类

| 类名           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| BpRefBase      | RefBase的子类，提供remote()方法获取远程Binder                |
| IInterface     | Binder服务接口的基类，Binder服务通常需要同时提供本地接口和远程接口 |
| BpInterface    | 远程接口的基类，远程接口是供客户端调用的接口集               |
| BnInterface    | 本地接口的基类，本地接口是需要服务中真正实现的接口集         |
| IBinder        | Binder对象的基类，BBinder和BpBinder都是这个类的子类          |
| BpBinder       | 远程Binder，这个类提供transact方法来发送请求，BpXXX实现中会用到 |
| BBinder        | 本地Binder，服务实现方的基类，提供了onTransact接口来接收请求 |
| ProcessState   | 代表了使用Binder的进程                                       |
| IPCThreadState | 代表了使用Binder的线程，这个类中封装了与Binder驱动通信的逻辑 |
| Parcel         | 在Binder上传递的数据的包装器                                 |

- IBinder：描述了所有在Binder上传递的对象，它既是本地对象`BBinder`的父类，也是远程对象`BpBinder`的父类。

- BpBinder：远程Binder，客户端调用，其`handle`方法会返回指向Binder服务实现者的句柄，其`transact`方法会将远程调用的参数封装好发送给Binder驱动。

  > 由于每个Binder服务通常都会提供多个服务接口，而`transact`方法中的`uint32_t code`参数就是用来对**服务接口**进行编号区分的。Binder服务的每个接口都需要指定一个唯一的`code`，这个`code`要在`Proxy`和`Native`端配对好。当客户端将请求发送到服务端的时候，服务端根据这个`code`（`onTransact`方法中）来区分调用哪个接口方法。

- BBinder：本地Binder，服务的提供方，所有Binder服务的实现者都需要继承该类，重写`onTransact`方法，该方法是所有请求的入口，与BpBinder中的`transact`方法对应。

- IXXX：每个Binder服务都是为某个功能实现的，因此其本身会定义一套接口集（通常是C++的一个类）来描述提供的所有功能，而为了开发方便，Binder服务实现自身的类与提供客户端调用的类会有一个公共描述服务接口的基类(IXXX)来继承，同时两个类中的接口应当是一致的。

- `BnXXX::onTransact`方法的职责是根据请求的`code`区分具体调用的是哪个接口，然后按照顺序（`BpXXX`中写入的顺序）从`Parcel`中读出打包好的参数，接着调用留待子类实现的虚方法。【`XXXService`类才是服务实现的真正本体】

- IInterface：`IXXX`通常是`IInterface`的子类，`IInterface`中定义了`onAsBinder`让子类实现，`onAsBinder`在本地对象的实现类中会返回本地对象，在远程对象的实现类中会返回远程对象。`onAsBinder`又会被`asBinder`调用，如此在代码中就不用区分是本地对象还是远程对象，直接可通过`IXXX:asBInder`获得，这在跨进程传递`Binder`对象时起了很大作用。

- BpInterface、BnInterface都是模板类：

  ```c++
  // IInterface.h
  class IInterface: public virtual RefBase {
      public:
      	IInterface();
      	static sp<IBinder> asBinder(const IInterface*);
      	static sp<IBinder> asBinder(const sp<IInterface>&);
      protected:
      	virtual ~IInterface();
      	virtual IBinder* onAsBinder() = 0;
  }
  
  template<typename INTERFACE>
  class BnInterface: public INTERFACE, public BBinder {
  	public:
  		virtual sp<IInterface> queryLocalInterface(const String16& _descriptor);
  		virtual const String16& getInterfaceDescriptor() const;
  	protected:
  		virtual IBinder* onAsBinder();
  };
  
  template<typename INTERFACE>
  class BpInterface: public INTERFACE, public BpRefBase {
  	public:
  		BpInterface(const sp<IBinder>& remote);
  	protected:
  		virtual IBinder* onAsBinder();
  };
  ```

  这两个类在继承自`INTERFACE`(Binder服务接口的基类:`IXXX`)的基础上各自继承了另一个类。`BnInterface`继承了`BBinder`类，由此可以复写`onTransact`方法来提供实现。`BpInterface`继承了`BpRefBase`，通过该类的`remote`方法可以获取到指向服务实现方的句柄。在客户端接口的实现类中，每个接口在组装好参数之后，都会调用`remote() -> transact`来发送请求，而这里其实就是调用的`BpBinder`的`transact`方法，这样请求便通过`Binder`到达了服务实现方的`onTransact`中。过程图如下:

  ![libbinder中的调用过程](/resource/libbinder中的调用过程.png)

  > 另：为了区分**服务**，每个Binder服务会在类中定义`descriptor`字符串常量来作为唯一的标识，并通过`getInterfaceDescriptor`方法返回该常量。为了便于调用者获得到调用接口，服务接口的公共基类需提供一个`asInterface`方法来返回基类对象指针。
  >
  > `libbinder`中通过宏来实现以上逻辑，简化每个Binder服务都要实现的麻烦。开发者只要在接口基类(`IXXX`)头文件中使用`DECLARE_META_INTERFACE`宏便完成了需要的组件的声明。然后在cpp文件中使用`IMPLEMENT_META_INTERFACE`便完成了这些组件的实现。

  ```c++
  // IInterface.h
  #define DECLARE_META_INTERFACE(INTERFACE)
  	static const android::String16 descriptor;
  	static android::sp<I##INTERFACE> asInterface(const android::sp<android::IBinder>& obj);
  	virtual const android::String16& getInterfaceDescriptor() const;
  	I##INTERFACE();
  	virtual ~I##INTERFACE();
  
  #define IMPLEMENT_META_INTERFACE(INTERFACE, NAME)
  	const android::String16 I##INTERFACE::descriptor(NAME);
      const android::String16& I##INTERFACE::getInterfaceDescriptor() const {
          return I##INTERFACE::descriptor;
      }
      android::sp<I##INTERFACE> I##INTERFACE::asInterface(const android::sp<android::IBinder>& obj) {
          android::sp<I##INTERFACE> intr;
          if(obj != NULL) {
              intr = static_cast<I##INTERFACE*>(obj -> 
                                                queryLocalInterface(I##INTERFACE::descriptor).get());
              if(intr == NULL) {
                  intr = new Bp##INTERFACE(obj);
              }
          }
          return intr;
      }
      I##INTERFACE::I##INTERFACE() {}
      I##INTERFACE::~I##INTERFACE() {}
  ```

  `asInterface`方法逻辑：

  - 先尝试通过`queryLocalInterface`获取本地Binder，如果是在服务所在的进程中调用，便能获得到本地Binder，否则返回`NULL`；
  - 如果获取不到本地Binder，则创建并返回一个远程Binder。由此保证了：我们在进程内部的调用，是直接通过方法调用的形式；而如果不在同一个进程的时候，是通过Binder进行跨进程的调用。

- ProcessState：封装了Binder驱动的`open`和`mmap`方法，该类是一个单例类，通过`self()`接口获取实例，一个<font color='red'>进程</font>只有一个实例类，其构造方法初始化`mDriverFD`时调用了`open_driver`方法，在函数体中通过`mmap`进行了内存映射。

    - `open_driver`函数：
      - 首先通过`open`系统调用打开了`dev/binder`设备。
      - 后通过`ioctl`获取Binder实现的版本号，并检查是否匹配。
      - 再通过`ioctl`设置进程支持的最大线程数量。

    - Binder传递数据大小限制：

      由于Binder数据需跨进程传递，且需在内核上开辟空间，故允许在Binder上传递的数据有大小限制（通过`mmap`指定）。

      ```c++
      // ProcessState.cpp
      #define BINDER_VM_SIZE ((1*1024*1024) - (4096*2))	// 1MB - 8KB = 1016KB
      mVMStart = mmap(0, BINDER_VM_SIZE, PROT_READ, MAP_PRIVATE | MAP_NORESERVE, mDriverFD, 0);
      ```

    > Tip：通过`showmap`命令可以看到进程的详细内存占用情况。

- IPCThreadState：负责与Binder驱动通信，该类是一个单例类，一个<font color='red'>线程</font>只有一个实例。`BpBinder::transact`方法在发送请求时，其实就是调用了`IPCThreadState::self()->transact()`。

    该类重要方法如下：

    | 方法                 | 说明                                            |
    | -------------------- | ----------------------------------------------- |
    | transact             | 公开接口，供Proxy发送数据到驱动，并读取返回结果 |
    | sendReply            | 供Server端写回请求的返回结果                    |
    | waitForResponse      | 发送请求后等待响应结果                          |
    | talkWithDriver       | 通过ioctl BINDER_WRITE_READ来与驱动通信         |
    | writeTransactionData | 写入一次事务的数据                              |
    | executeCommand       | 处理binder_driver_return_protocol协议命令       |
    | freeBuffer           | 通过BC_FREE_BUFFER命令释放Buffer                |

- Parcel数据包装类

    - Parcel提供了所有基本类型和Binder对象的写入和读出接口<i style='color:grey;font-size:15px;'>[实现原理即上面提到的驱动传递对象]</i>（非基本数据类型需开发者进行拆分），调用者可以以任意顺序往里放入需要的数据，所有写入的数据就像是被打成一个整体的包，可以直接在Binder上传输。

        ```c++
        // Parcel.cpp
        status_t Parcel::writeStrongBinder(const sp<IBinder>& val) {
            return flatten_binder(ProcessState::self(), val, this);
        }
        ```

    - `Parcel`既包含C++部分的实现，同时也提供了Java的接口，中间通过JNI连接。Java层的接口其实仅是一层包装，真正的实现都位于C++中。

##### Framework层的线程管理

`ProcessState::setThreadPoolMaxThreadCount`方法中，通过`BINDER_SET_MAX_THREADS`命令设置进程支持的最大线程数量，由此驱动便知该Binder服务支持的最大线程数。驱动在运行过程中，会根据需要，并在没有超过上限的情况下，通过`BR_SPAWN_LOOPER`命令通知进程创建线程。

Binder中线程创建过程：

- 驱动发出`BR_SPAWN_LOOPER`命令。
- `IPCThreadState`收到`BR_SPAWN_LOOPER`请求，在`executeCommand`方法中调用`ProcessState::spawnPooledThread`为线程设定名称并创建。
- 线程`run`后，调用`ProcessState::threadLoop`方法，其内部调用`IPCThreadState::joinThreadPool()`方法将自身添加到线程池中。
- 在`IPCThreadState::joinThreadPool()`方法中，会根据当前线程是否是主线程`BC_ENTER_LOOPER`或者`BC_REGISTER_LOOPER`命令告知驱动线程已经创建完毕。

![Binder中的线程创建](/resource/Binder中的线程创建.png)

##### 服务的发布

服务实现完成后，必须要注册到`ServiceManager`中才能被其他模块获取和使用，在`BinderService`类中，提供了`publishAndJoinThreadPool`方法来简化服务的发布。

```c++
// BinderService.h
static void publishAndJoinThreadPool(bool allowIsolated = false) {
    publish(allowIsolated);
    joinThreadPool();
}

static status_t publish(bool allowIsolated = false) {
    sp<IServiceManager> sm(defaultServiceManager());
    return sm->addService(
        String16(SERVICE::getServiceName()),
    	new SERVICE(), allowIsolated);
}

static void joinThreadPool() {
    sp<ProcessState> ps(ProcessState::self());
    ps->startThreadPool();
    ps->giveThreadPoolName();
    IPCThreadState::self()->joinThreadPool();
}
```

由此，`Binder`服务的发布步骤：

- 通过`IServiceManager::addService`在`ServiceManager`中进行服务的注册。
- 通过`ProcessState::startThreadPool`启动线程池。
- 通过`IPCThreadState::joinThreadPool`将主线程加入`Binder`中。

##### 服务的获取

<font color='red'>以`PowerManager`为例：</font>

客户端应通过`BpPowerManager`对象来请求服务，`BpPowerManager`的构造函数需要一个`sp<IBinder>`类型参数。

```c++
// IPowerManager.cpp
BpPowerManager(const sp<IBinder>& impl) : BpInterface<IPowerManager>(impl) {}
```

该参数可通过ServiceManager中的两个方法获取。其中getService是阻塞式的，checkService是非阻塞式的，传递的参数即为addService时对应的字符串。

```c++
// IServiceManager.h
// Retrieve an existing service, blocking for a few seconds if it doesn't yet exist.
virtual sp<IBinder> getService(const String16& name) const = 0;

// Retrieve an existing service, non-blocking.
virtual sp<IBinder> checkService(const String16& name) const = 0;
```

但直接调用`BpPowerManager`的构造方法，只适用于客户端与`PowerManager`服务所在进程非同一进程时候。针对该问题，Binder Framework提供了`interface_cast`方法来获取服务的接口对象，该方法是一个模板方法，其本身可根据是否是在同一进程，而自动确定返回本地Binder还是远程Binder。<i style='color:grey;font-size:15px;'>[实现原理即上面提到的`DECLARE_META_INTERFACE`宏文件]</i>

```c++
// IInterface.h
template<typename INTERFACE>
inline sp<INTERFACE> interface_cast(const sp<IBinder>& obj) {
	return INTERFACE::asInterface(obj);
}
```

故获取`PowerManager Binder`接口对象如下：

```c++
// PowerHAL.cpp
const String16 serviceName("power");
sp<IBinder> bs = defaultServiceManager()->checkService(serviceName);
if (bs = NULL) {
    return NAME_NOT_FOUND;
}
sp<IPowerManager> pm = interface_cast<IPowerManager>(bs);
```

##### C++层的ServiceManager

`ServiceManager`的具体实现，该类是一个独立的可执行文件，在设备中的进程名称是`/system/bin/servicemanager`。这个也是其可执行文件的路径。

`service_manager.c`中的实现与普通Binder服务的实现不同，其并未通过继承接口类实现，而是通过自行编写`binder.c`直接和Binder驱动通信。`service_manager.c`主要方法如下：

| 方法名称        | 方法说明                                          |
| --------------- | ------------------------------------------------- |
| main            | 可执行文件入口函数                                |
| svcmgr_handler  | 请求的入口函数，类似于普通Binder服务的onTransanct |
| do_add_service  | 注册一个Binder服务                                |
| do_find_service | 通过名称查找一个已经注册的Binder服务              |

- `main`方法

  ```c++
  // service_manager.c
  int main() {
      struct binder_state *bs;
      bs = binder_open(128*1024);
      if(!bs) {
          ALOGE("failed to open binder driver\n");
          return -1;
      }
      
      if(binder_become_context_manager(bs)) {
          ALOGE("cannot become context manager (%s)\n", strerror(errno));
          return -1;
      }
      ...
      binder_loop(bs, svcmgr_handler);
      
      return 0;
  }
  ```

  - `binder_open(128*1024)`是打开Binder，并指定缓存大小为128KB，由于`ServiceManager`提供的接口很简单，因此并不需要普通进程那么多（1MB-8KB）的缓存。
  - `binder_become_context_manager(bs)`使自己成为Context Manager。这里的Context Manager是Binder驱动里面的名称，等同于`ServiceManager`。`binder_become_context_manager`的方法实现只有一行代码：`ioctl(bs->fd, BINDER_SET_CONTEXT_MGR, 0)`，指定该进程为`ServiceManager`（即binder服务的管家）。
  - `binder_loop(bs, svcmgr_handler)`是在Looper上循环，等待其他模块请求服务，处理client端发来的请求。

- `ServiceManager`中，通过`svcinfo`结构体来描述已注册的Binder服务，所有的服务通过`next`指针形成了一个链表，`handle`指向由Binder驱动翻译过的Binder服务的句柄（实体），`name`为服务的名称。

  ```c
  // service_manager.c
  struct svcinfo {
      struct svcinfo *next;
      uint32_t handle;
      struct binder_death death;
      int allow_isolated;
      size_t len;
      uint16_t name[0];
  }
  ```


#### Java部分

Binder Framework Java层到C++层结构：

![Binder Framework Java层到C++层结构](/resource/Java层到C++层结构.png)

JNI：Java Native Interface，Java虚拟机提供的机制，该机制使得Native代码可以和Java代码相互通信。

<font color='red'>以`ActivityManager`（管理Android系统的四大组件）实现为例：</font>

`ActivityManager`实现结构：

![ActivityManager实现结构](/resource/ActivityManager实现结构.png)

下面是上图中几个类的说明：

| 类名                   | 说明                   |
| ---------------------- | ---------------------- |
| IActivityManager       | Binder服务的公共接口   |
| ActivityManagerProxy   | 供客户端调用的远程接口 |
| ActivityManagerNative  | Binder服务实现的基类   |
| ActivityManagerService | Binder服务的真正实现   |

我们在通过`ActivityManager`调用内部接口，如`getMemoryInfo`时，

```java
// ActivityManager.java
public void getMemoryInfo(MemoryInfo outInfo) {
    try {
        ActivityManagerNative.getDefault().getMemoryInfo(outInfo);
    } catch(RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}
```

该方法调用了`ActivityManagerNative.getDefault()`中的方法，其实就是在`IActivityManager`中通过`"activity"`服务标识获取到`ActivityManager`的Binder对象。

```java
// ActivityManagerNative.java
static public IActivityManager getDefault() {
    return gDefault.get();
}

private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
    protected IActivityManager create() {
        IBinder b = ServiceManager.getService("activity");
        if (false) {
            Log.v("ActivityManager", "default service binder = " + b);
        }
        IActivityMangager am = asInterface(b);
        if (false) {
            Log.v("ActivityManager", "default service = " + am);
        }
        return am;
    }
};

static public IAcitvityManager asInterface(IBinder obj) {
    if (obj == null) {
        return null;
    }
    IActivityManager in = (IActivityManager) obj.queryLocalInterface(descriptor);
    if (in != null) {
        return in;
    }
    
    return new ActivityManagerProxy(obj);
}
```

然后通过`queryLocalInterface`确定有没有本地Binder，如果有则直接返回，否则创建一个`ActivityManagerProxy`对象。很显然，假设在`ActivityManagerService`所在的进程中调用这个方法，则`queryLocalInterface`会直接返回本地Binder；而假设在其他进程中调用，该方法返回为空，由此导致其他调用获取到的对象其实就是`ActivityManagerProxy`，之后再通过Binder跨进程调用`ActivityManagerService`中的方法。（这里的`asInterface`方法的实现与C++层类似）

#### AIDL

Android Interface Definition Language，是SDK提供的一种机制，应用通过该机制可以提供跨进程的服务供其他应用使用。

开发一个基于AIDL的Service的步骤：

- 定义一个`.aidl`文件（使用Java语法定义，每个`.aidl`文件只能包含一个interface，并且要包含interface的所有方法声明）
- 实现接口。[即之后自动生成的`Stud()`]
- 暴露接口给客服端使用。[即之后自动生成的`Proxy`]

一个`.aidl`文件的示例：

```java
// IRemoteService.aidl
package com.example.android;

interface IRemoteService {
    int getPid();
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString);
}
```

包含`.aidl`文件的工程，经编译生成文件结构如下：

![AIDL生成的Java结构](/resource/AIDL生成的Java结构.png)

在这个生成的Java文件中，包括：

- 一个名称为`IRemoteService`的`interface`，该`interface`继承自`android.os.IInterface`并且包含了我们在`.aidl`文件中声明的接口方法。

- `IRemoteService`中包含了一个名称为`Stub`的静态内部类，这个类是一个抽象类，它继承自`android.os.Binder`并且实现了`IRemoteService`接口。这个类中包含了一个`onTransact`方法。

- `Stub`内部又包含了一个名称为`Proxy`的静态内部类，`Proxy`类同样实现了`IRemoteService`接口。

- 下面列举一下各层几个类的对应情况：

  | C++层 | Java层    | AIDL            |
  | ----- | --------- | --------------- |
  | BpXXX | XXXProxy  | IXXX.Stub.Proxy |
  | BnXXX | XXXNative | IXXX.Stub       |

  - `Stub`是提供给开发者实现业务的父类，`Proxy`实现了对外提供的接口。`Stub`和`Proxy`两个类都有一个`asBinder`的方法。

    `Stub`类中`asBinder`实现就是返回自身对象。

    ```java
    @Override
    public android.os.IBinder asBinder() {
        return this;
    }
    ```

    `Proxy`类中`asBinder`实现是返回构造函数中获取的`mRemote`对象。（远程服务在当前进程的标识）

    ```java
    private android.os.IBinder mRemote;
    
    Proxy(android.os.IBinder remote) {
        mRemote = remote;
    }
    
    @Override
    public android.os.IBinder asBinder() {
        return mRemote;
    }
    ```

  - 进程间通信时，在`Stub.Proxy`中通过`Parcel`对象及`transact`调用对应远程服务的接口，而在`Stub`类中，生成的`onTransact`方法对应处理这个请求。（根据`code`区分请求的接口，通过`data`获取请求的参数，调用由子类实现的抽象方法）

    ```java
    @Override
    public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) 
    	throws android.os.RemoteException {
        switch(code) {
            case INTERFACE_TRANSACTION:
                reply,writeStrng(DESCRIPTION);
                return true;
            ...
        }
        return super.onTransact(code, data, reply, flags);
    }
    ```
