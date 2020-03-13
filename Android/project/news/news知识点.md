- 页面架构？

  - mvc

  - mvp

  - mvvm (通过DataBinding解决mvp中interface过多的问题，同时实现双向绑定)
- 模块化 层次化 控件化(组件化)
- 多用自定义View实现解耦。

## 基本配置

模块化版本号处理(project `build.gradle`)

```groovy
project.ext.set("minSdkVersion", "21")
project.ext.set("targetSdkVersion", "28")
project.ext.set("compileSdkVersion", "28")
project.ext.set("buildToolsVersion", "28.0.3")
project.ext.set("versionCode", "10001")
project.ext.set("versionName", "1.0.0.01")

project.ext.set("androidXVersion", "1.0.0")
project.ext.set("constraintlayoutVersion", "1.1.3")
```

引入Androidx(app `build.gradle`)

```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion Integer.parseInt(project.compileSdkVersion)
    buildToolsVersion project.buildToolsVersion

    defaultConfig {
        applicationId "com.example.news"
        minSdkVersion Integer.parseInt(project.minSdkVersion)
        targetSdkVersion Integer.parseInt(project.targetSdkVersion)
        versionCode Integer.parseInt(project.versionCode)
        versionName project.versionName
//        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation "androidx.appcompat:appcompat:$project.androidXVersion"
    implementation "androidx.constraintlayout:constraintlayout:$project.constraintlayoutVersion"
    implementation project(":base")
}
```

- gradle是一个编程框架，gradle使用的语言是groovy。

- test包括两种：单元测试和机器上的测试。

- gradle下载慢：

  - 引用阿里云

    ```groovy
    buildscript {
        
        repositories {
            maven {
                url  'https://maven.aliyun.com/repository/google'
            }
            maven {
                url 'https://maven.aliyun.com/repository/jcenter'
            }
            maven {
                url 'https://maven.aliyun.com/repository/public'
            }
            google()
            jcenter()
            maven {
                url 'https://maven.google.com/'
                name 'Google'
            }
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:3.1.4'
        }
    }
    
    allprojects {
        repositories {
            maven {
                url  'https://maven.aliyun.com/repository/google'
            }
            maven {
                url 'https://maven.aliyun.com/repository/jcenter'
            }
            maven {
                url 'https://maven.aliyun.com/repository/public'
            }
            google()
            jcenter()
        }
    }
    ```

  - 自主搭建服务器

- 搭建本地私有私有服务器

  - Nexus(不能放iOS，cocas，cocopods的库)
  - artifactory(推荐)

- 命令行编译

  ```
  // 移除gradle
  rm -rf ~/.gradle
  // 编译
  gradlew assembleDebug [--info]
  // 清除项目
  gradlew clean
  // 卸载应用
  // -k表示不删除程序运行所产生的数据和缓存目录，如数据库文件
  adb uninstall [-k] com.example.news
  // 安装应用
  // -r表示替换到原有apk
  adb install [-r] app/build/outputs/apk/debug/app-debug.apk
  ```

- 开启dataBinding(双向数据绑定)，减少不必要的findViewById。

  ```groovy
  dataBinding {
      enabled true
  }
  ```

- 使用dataBinding

  布局被`<layout>`标签标识，dataBinding名字规则如：`activity_main --> ActivityMainBinding`

  ```
  mBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
  ```

- 引用系统资源方式

  - `android:layout_height="?attr/actionBarSize"`
  - `android:background="@android:color/white"`

- 兼容java8

  ```groovy
  compileOptions {
      sourceCompatibility JavaVersion.VERSION_1_8
      targetCompatibility JavaVersion.VERSION_1_8
  }
  ```

## BottomNavigationView

  - 选中的图标不使其变大：`app:labelVisibilityMode="labeled"`

  - 改变标题字体大小

    - 重写属性

    ```xml
    // dimens.xml
    <dimen name="design_bottom_navigation_active_text_size" tools:override="true">10sp</dimen>
        <dimen name="design_bottom_navigation_text_size" tools:override="true">10sp</dimen>
    ```

    - 重写布局

    ```xml
    <com.google.android.material.bottomnavigation.BottomNavigationView
            ...
          	app:itemTextAppearanceActive="@style/BottomNavigationView.Active"
            app:itemTextAppearanceInactive="@style/BottomNavigationView"
            ... />
    <!-- styles -->
    <style name="BottomNavigationView" parent="@style/TextAppearance.AppCompat.Caption">
        <item name="android:textSize">10sp</item>
    </style>
    
    <style name="BottomNavigationView.Active" parent="@style/TextAppearance.AppCompat.Caption">
        <item name="android:textSize">11sp</item>
    ```
  - 反射解决BottomNavigationView大于三个时不显示text的问题（<font color='red'>该坑已被官方填上[因混淆后反射可能无``mShiftingMode``属性]，用`labelVisibilityMode`属性代替，support 28.0及以上版本可忽略</font>），BottomNavigationView最多只能有5个。

    ```java
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        disableShiftMode(mBinding.bottemView);
    }
    
    @SuppressLint("RestrictedApi")
    private void disableShiftMode(BottomNavigationView view) {
        BottomNavigationMenuView menuView = (BottomNavigationMenuView) view.getChildAt(0);
        try {
            Field shiftingMode = menuView.getClass().getDeclaredField("mShiftingMode");
            shiftingMode.setAccessible(true);
            shiftingMode.setBoolean(menuView, false);
            shiftingMode.setAccessible(false);
    
            for (int i = 0; i < menuView.getChildCount(); i++) {
                BottomNavigationItemView item = (BottomNavigationItemView) menuView.getChildAt(i);
                item.setShiftingMode(false);
                item.setChecked(item.getItemData().isChecked());
            }
        } catch (NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }
    ```

## 安装artifactory

- 解压artifactory-6.6.0.zip

- 命令行执行 `java -jar artifactory-injector-1.1.jar`
  - 选择 2 `Inject artifactory`
  - 复制home目录：例如`C:\Softwares\artifactory-pro-6.6.0`
  - 提示找到artifactory，输入`yes`进行破解
  - 选择 1 `generate License String`
  - 输入exit退出

- 新开命令行 搭建本地仓库
  - 切换到bin目录下，执行`artifactory.bat start`。
  - 网页输入`localhost:8081/artifactory`，会自动跳转到`localhost:8081/artifactory/webapp/#/home`
  - 复制得到的`License`，点击下一步。
  - 设置管理员密码。
  - 出现设置代理界面`Configure a ProxyServer`，直接跳过，继续跳过创建仓库。

- 创建远程代理仓库

  - Admin -> Remote -> new -> maven 命名
  - 配置remote，然后配置虚拟分组virtual。

- 配置

  - 代理是可以分组的。

  - 客户端使用本地仓库

    ```groovy
    buildscript {
        repositories {
            maven {
                url  'http://localhost:8081/artifactory/android_group/'
            }
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:3.1.4'
        }
    }
    
    allprojects {
        repositories {
            maven {
                url  'http://localhost:8081/artifactory/android_group/'
            }
        }
    }
    ```

    ```properties
    distributionUrl=http\://localhost:8081/artifactory/android_local/gradle-5.4.1-all.zip
    ```

## Toolbar

## ActionBar

- setHomeButtonEnabled： 小于4.0版本的默认值为true的。但是在4.0及其以上是false，该方法的作用：决定左上角的图标是否可以点击。

- setDisplayShowHomeEnabled：使左上角图标是否显示。

- setDisplayHomeAsUpEnabled：给左上角图标的左边加上一个返回的图标 。

- setDisplayShowCustomEnabled：使自定义的普通View能在title栏显示。


## CollapsingToolbarLayout

`app:contentScrim="?attr/colorPrimary"`：ToolBar被折叠到顶部固定时候的背景。

## CoordinatorLayout

- CoordinatorLayout继承自ViewGroup，是一个FrameLayout。

- `app:layout_scrollFlags`

  - scroll：所有想滚动出屏幕的view都需要设置这个flag， 没有设置这个flag的view将被固定在屏幕顶部。
  - enterAlways：设置这个flag时，向下的滚动都会导致该view变为可见。
  - enterAlwaysCollapsed：当你的视图已经设置minHeight属性又使用此标志，向下滚动时，你的视图只能已minHeight高度进入，只有当滚动视图到达顶部时才扩大到完整高度。
  - exitUntilCollapsed：这里也涉及到最小高度。发生向上滚动事件时，Child View向上滚动退出直至最小高度，然后Scrolling View开始滚动。也就是，Child View不会完全退出屏幕。
  - snap：视图在滚动时会有一种“就近原则”。当视图展开时，如果滑动中展开的内容超过视图的75%，那么视图依旧会保持展开；当视图处于关闭时，如果滑动中展开的部分小于视图的25%，那么视图会保持关闭。总的来说，就是会让动画有一种弹性的视觉效果。

- `app:layout_collapseMode`：设置折叠效果

  - pin：固定模式，在折叠的时候最后固定在顶端。
  - parallax：视差模式，在折叠的时候会有个视差折叠的效果。

- `android:layout_anchor `、`android:layout_anchorGravity`：设置于其他视图关联在一起的悬浮视图（如 FloatingActionButton）。

### Behavior

- Behavior就是一个应用于View的观察者模式，一个View跟随着另一个View的变化而变化。

  在Behavior中，被观察View(事件源)称为`denpendcy`，观察者View称为`child`。

  `CoordinatorLayout.Behavior<>`的泛型代表观察者View，即`child`。

- CoordinatorLayout的工作原理是搜索定义了CoordinatorLayout Behavior 的子view，不管是通过在xml中使用app:layout_behavior标签还是通过在代码中对view类使用@DefaultBehavior修饰符来添加注解。当滚动发生的时候，CoordinatorLayout会尝试触发那些声明了依赖的子view。

- 两个基本方法

  ```java
  // 返回false表示child不依赖dependency，ture表示依赖。该方法通常被用来指定被观察者
  public boolean layoutDependsOn(CoordinatorLayout parent, T child, View dependency) {...}
  // 当dependency发生改变时（位置、宽高等），执行该函数.返回true表示child的位置或者是宽高要发生改变，否则就返回false
  public boolean onDependentViewChanged(CoordinatorLayout parent, T child, View dependency) {...}
  ```

### 嵌套滚动

## 插件化配置

非主app模块：`apply from: rootProject.file('cc-settings-2.gradle')`

主模块：

```groovy
ext.mainApp = true
apply from: rootProject.file('cc-settings-2.gradle')
```

## 显示Fragment

```java
// 显示Fragment
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
transaction.replace(R.id.container, mHomeFragment);
transaction.commit();
```



