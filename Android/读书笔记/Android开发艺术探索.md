## Activity生命周期
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

- 资源内存不足导致低优先级低的Activity被杀死
	- Activity优先级：前台Activity -> 可见但非前台Activity -> 后台Activity
	- 如果一个进程中没有四大组件在执行，那么该进程将很快被系统杀死。

#### configChanges的常用参数和含义
- locale：指切换了系统语言
- orientation：指屏幕方向发生了改变
- keyboardHidden：键盘的可访问性发生了改变，比如用户调出了键盘
- screenSize：api13 新添加，指旋转屏幕时
- smallestScreenSize：api13新添加，指用户切换到了外部显示设备等。
>当不想让Activity在屏幕旋转时重新创建，可加
>android:configChanges="orientation|screenSize"

## Activity启动模式（launchMode）
- standard：标准模式，一个任务栈中可以有多个实例，每个实例可以属于不同的任务栈。该种模式下，谁启动了这个Activity，该Activity就运行在启动它的Activity所在栈中。
- singleTop：栈顶复用模式。该模式下，如果新的Activity已位于任务栈栈顶，那么该Activity不会被重新创建，同时会回调它的onNewIntent()方法。
- singleTask：栈内复用模式。单实例模式，只要Activity在一个栈中存在，那么多次启动此Activity都不会重新创建实例，同时会将该活动调到栈顶并回调它的onNewIntent()方法，并且由于singleTask默认具有clearTop效果，该活动调到栈顶时会导致在该活动上的所有活动出栈。
- singleInstance：单实例模式。具有singleTask模式的所有特性，同时具有此模式的Activity只能单独位于一个任务栈中。

#### Activity所需的任务栈
- TaskAffinity(任务相关性)：该参数标识了一个Activity所需任务栈的名字，默认情况下，所有Activity所需任务栈的名字为应用的包名。
- TaskAffinity属性主要与singleTask启动模式或者allowTaskReparenting属性配对使用。
- 指定启动模式的方式：
	- 通过AndroidMenifest指定。
	```xml
	<activity
	   android:launchMode="singleTask"
     android:label="@string/app_name" />
	```
	- 通过Intent设置标志位指定（相比于第一种方式，该方式优先级更高）
	```java
	Intent intent = new Intent();
	intent.setClass(MainActivity.this, SecondActivity.class);
	intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
	startActivity(intent);
	```


