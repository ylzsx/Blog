#### Android控件架构
- Android中，控件大致分为两类，即ViewGroup控件和View控件。其中，ViewGroup控件作为父控件可以包含多个View控件。
- 控件树
	- 每棵控件树的顶部都是一个ViewParent对象，它是整棵树的控制核心，所有的交互事件都由它统一调度和分配。
	- findViewById()方式就是在控件树中以树的**深度**优先遍历来查找对应元素。
- Android界面架构
	- Activity -> PhoneWindow -> DecorView -> TitleView和ContentView。
	- 每个Activity都包含一个Window对象，其通常由PhoneWindow实现。
	- PhoneWindow下将一个DecorView设置为整个应用窗口的跟View，DecorView中所有的View监听事件都通过WindowManagerService进行接收，并通过Activity对象来回调相应的onClickListener方法。
	- DecorView将屏幕分为两个部分，一个是TitleView，一个是ContentView。其中，ContentView是ID为content的FrameLayout。而activity_main.xml就是设置在该FrameLayout中。
	- 用户可以通过requestWindowFeature(Window.FEATURE_NO_TITLE)来设置全屏显示。但该方法必须在setContentView()前调用才能生效。
	- 在程序中，当setContentView()调用后，ActivityManagerService会回调onResume()方法完成界面的绘制。

#### Activity生命周期
**1. 典型情况下的生命周期**

 - 对于一个Activity,第一次启动，回调如下： onCreate -> onStart -> onResume
 - 当用户打开新的Activity或者切换到桌面时，回调如下：onPause -> onStop。若新Activity采用了透明主题，则不会回调onStop。
 - 当用户再次回到原Activity时，回调如下：onRestart -> onStart -> onResume。
 - 当用户按返回键时，回调如下：onPause -> onStop -> onDestroy。
 - onStart和onStop是从Activity是否可见这个角度来回调的，而onResume和onPause是从Activity是否位于前台这个角度来回调的。
 - 当前Activity A关闭，Activity B开启时，回调如下：onPause -> onCreate -> onStart -> onResume -> onStop。（故不能在onPause中做重量级的操作，因为必须onPause执行完成后新的Activity才能Resume）
   
**2. 异常情况下的生命周期**

- 资源相关的系统配置发生改变导致Activity被杀死并重新创建（eg. 旋转屏幕）
	- Activity在异常情况下终止时，系统会调用onSaveInstanceState来保存当前的Activity状态。该方法在onStop前调用，与onPause方法调用时机无法确定。
	- 当Activity被重新创建后，系统会调用onRestoreInstanceState，并且将Activity销毁时onSaveInstanceState保存的Bundle对象作为参数传递给onRestoreInstanceState和onCreate方法。该方法的调用时间在onStart之后。
	- onSaveInstanceState和onRestoreInstanceState方法，系统只在Activity异常终止时才会调用，即系统只有在Activity被销毁并且有机会重新显示的情况下才会去调用。

- 资源内存不足导致低优先级的Activity被杀死