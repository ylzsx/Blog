#### Android控件架构
- Android中，控件大致分为两类，即ViewGroup控件和View控件。其中，ViewGroup控件作为父控件可以包含多个View控件。
- 控件树
	- 每棵控件树的顶部都是一个ViewParent对象，它是整棵树的控制核心，所有的交互事件都由它统一调度和分配。
	- findViewById()方式就是在控件树中以树的**深度**优先遍历来查找对应元素。
- Android界面架构
	- 每个Activity都包含一个Window对象，其通常由PhoneWindow实现。
	- PhoneWindow下将一个DecorView设置为整个应用窗口的跟View，DecorView中所有的View监听事件都通过WindowManagerService进行接收，并通过Activity对象来回调相应的onClickListener方法。
	- DecorView将屏幕分为两个部分，一个是TitleView，一个是ContentView。其中，ContentView是ID为content的FrameLayout。而activity_main.xml就是设置在该FrameLayout中。
	- 用户可以通过requestWindowFeature(Window.FEATURE_NO_TITLE)来设置全屏显示。但该方法必须在setContentView()前调用才能生效。
	- 在程序中，当setContentView()调用后，ActivityManagerService会回调onResume()方法完成界面的绘制。

#### Activity生命周期