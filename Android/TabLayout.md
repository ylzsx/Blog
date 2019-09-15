### 自定义下换线长度

- 反射解决

```java
public class TabUtil {

    public static void setTabIndicatorLength(TabLayout tab, int leftDip, int rightDip) {
        Class<? extends TabLayout> tabClass = tab.getClass();
        Field tabStrip = null;
        try {
            tabStrip = tabClass.getDeclaredField("slidingTabIndicator");

            tabStrip.setAccessible(true);
            LinearLayout llTab = null;
            llTab = (LinearLayout) tabStrip.get(tab);

            int left = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, leftDip, Resources.getSystem().getDisplayMetrics());
            int right = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, rightDip, Resources.getSystem().getDisplayMetrics());

            for (int i = 0; i < llTab.getChildCount(); i++) {
                View child = llTab.getChildAt(i);
                child.setPadding(0,0,0,0);
                LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(0, LinearLayout.LayoutParams.MATCH_PARENT, 1);
                params.leftMargin = left;
                params.rightMargin = right;
                child.setLayoutParams(params);
                child.invalidate();
            }
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

    }
}
```

- 方式调用

```java
mTabLayout.post(new Runnable() {
    @Override
    public void run() {
        TabUtil.setTabIndicatorLength(mTabLayout, 70, 70);
    }
});
```

