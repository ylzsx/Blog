## Android中的13种Drawable

### ColorDrawable

- 如果在java中直接定义颜色，要加入`0x`，且不能把透明度漏掉。

  ```java
  int mycolor = 0xFF123456;
  btn.setBackgroundColor(mycolor);
  ```

- 使用系统定义好的Color

  ```
  android.background="@android:color/black"
  int getColor = Resource.getSystem().getColor(android.R.color.holo_green_light);
  ```

- 利用静态方法argb设置颜色

  - 在 xml 中设置颜色可以忽略透明度，但在java中必须指出透明度（省略的话表示完全透明）

  ```java
  txtShow.setBackgroundColor(Color.argb(0xff, 0x00, 0x00, 0x00));
  ```

### NiewPatchDrawable

### ShapeDrawable

- 根标签`shape`：`rectangle`、`oval`、`line`、`ring`
- size：图形宽高
- solid：背景颜色
- stroke：边框
- conner：圆角半径
- padding：边距
- gradient：渐变色 --> **type**：渐变类型,可选(**linear**线性,**radial**发散,**sweep**平铺)

### AnimationDrawable

```xml
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
                android:oneshot="false">
	<item android:drawable="@mipmap/ic_pull_to_refresh_loding01"
          android:duration="100"/>
    <item android:drawable="@mipmap/ic_pull_to_refresh_loding02"
          android:duration="100"/>
</animation-list>
```

```java
protected void onCreate(Bundle savedInstanceState) {
    ...
    Handler handler = new Handler();
    handler.postDelaed(new Runnable() {
        @OVerride
        public void run() {
            ad.start();
        }
    }, 300);
}
```

