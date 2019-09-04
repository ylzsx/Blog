####  更改EditText下划线颜色样式
- 需要自定义一个style
  ```xml
  <style name="MyEditText" parent="Theme.AppCompat.Light">
      <item name="colorControlNormal">@color/black</item>
      <item name="colorControlActivated">@color/yellow</item>
  </style>
  ```

- 使用android:theme引用样式
```xml
<EditText
    android:hint="@string/hint_account"
    android:inputType="phone"
    android:theme="@style/MyEditText"
    android:imeOptions="actionNext"
    android:layout_width="@dimen/width_s55"
    android:layout_height="wrap_content" />
```

#### EditText实现框输入
- ```android:gravity="top"```用来设置光标的起始位置从顶行开始，否则默认情况下是从中间行开始
- ```android:inputType="textMultiLine"```用来设置允许多行输入
```xml
<EditText
    android:gravity="top"
    android:inputType="textMultiLine"
    android:background="null"
    android:layout_width="match_parent"
    android:layout_height="@dimen/height_s20"/>
```

#### 出现软键盘，并同时上移布局
```xml
<activity android:name=".business.main.activity.PublishMessageActivity"
	android:windowSoftInputMode="stateVisible|adjustResize">
</activity>
```

#### 补充
- 取消EditText的下划线：```android:background="@null"```
- 