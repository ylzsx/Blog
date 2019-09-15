### ScrollView中嵌套RecyclerView的问题
**1. 问题**
- 页面滑动卡顿
- ScrollView滑动不正常
- RecyclerView内容显示不全
- 打开页面不显示在顶部

**2. 解决方案**
- 方案一（可解决RecyclerView内容不全和滑动卡顿的问题）
	1) 使用`NestedScrollView`代替`ScrollView`。
	2) 添加`mRecycleView.setNestedScrollingEnabled(false)`;

- 方案二（可解决打开页面不显示在顶部的问题）
  在NestedScrollView的第一个子布局加入如下代码：

  ```xml
  android:focusable="true"
  android:focusableInTouchMode="true"
  ```

### RecyclerView垂直水平嵌套解决外层卡顿问题

```java
mHomeRecyclerView.setHasFixedSize(true);
mHomeRecyclerView.setNestedScrollingEnabled(false);
mHomeRecyclerView.setItemViewCacheSize(200);
RecyclerView.RecycledViewPool recycledViewPool = new
    RecyclerView.RecycledViewPool();
mHomeRecyclerView.setRecycledViewPool(recycledViewPool);
```

