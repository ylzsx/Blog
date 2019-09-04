####  自定义CheckBox选中颜色
- 自定义一个style
```xml
<style name="MyCheckBox" parent="@android:style/Widget.Material.CompoundButton.CheckBox">
        <item name="android:colorControlActivated">@color/light_yellow</item>
        <item name="android:colorControlNormal">@color/grey</item>
</style>
```
- 使用android:theme引用样式

