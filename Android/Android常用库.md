####  常用库
**1. ButterKnife**

- 注解依赖
```
implementation 'com.jakewharton:butterknife:8.8.1'
annotationProcessor 'com.jakewharton:butterknife-compiler:8.8.1'
```
- 初始化【该方法必须在`setContent()`之后才能调用】
```java
ButterKnife.bind(this);
```

**2. Design Support**

```
implementation 'com.android.support:design:28.0.0'
implementation 'de.hdodenhof:circleimageview:2.1.0'
```

**3. recyclerview**
```
implementation 'com.android.support:recyclerview-v7:28.0.0'
```

**4. CardView**
```
implementation 'com.android.support:cardview-v7:28.0.0'
```

**5. Glide**

```xml
implementation 'com.github.bumptech.glide:glide:4.8.0'
annotationProcessor 'com.github.bumptech.glide:compiler:4.8.0'
```

- 在项目`build.gradle`中添加如下：

```
allprojects {
    repositories {
        mavenCentral()
        maven { url 'https://maven.google.com' }
        google()
        jcenter()
    }
}
```

- 在`AndroidManifest.xml`中添加`android:usesCleartextTraffic="true"`

**6. okhttp**

```
implementation 'com.squareup.okhttp3:okhttp:3.10.0'
```

- 在res下新增目录：`xml/network_security_config.xml`

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <network-security-config>
       <base-config cleartextTrafficPermitted="true" />
  </network-security-config>
  ```

- App的`AndroidManifest.xml`

  ```xml
  <application
      ...
  	android:networkSecurityConfig="@xml/network_security_config"
      ...
  />
  ```

- 加入

  ```gr
  compileOptions{
      sourceCompatibility JavaVersion.VERSION_1_8
      targetCompatibility JavaVersion.VERSION_1_8
  }
  ```

**7. LitePal**

- 依赖
```
implementation 'org.litepal.android:java:3.0.0'
```
- 官方网址：https://github.com/LitePalFramework/LitePal
- 初始化
  - 创建main/assets/litepal.xml
  - AndroidManifest.xml添加application中name标签
  ```xml
  <application
     android:name="org.litepal.LitePalApplication"
     ... >
  ```
  或者在onCreate中添加初始化
  ```java
  public void onCreate() {
      super.onCreate();
      LitePal.initialize(this);
  }
  ```

**8. CircleImageView**

```
implementation 'com.android.support:design:28.0.0'
implementation 'de.hdodenhof:circleimageview:2.1.0'
```

**9. Rxjava2**

```
implementation 'io.reactivex.rxjava2:rxjava:2.1.13'
implementation 'io.reactivex.rxjava2:rxandroid:2.0.2'
```

**10. Room**

```
implementation 'android.arch.persistence.room:runtime:1.1.1'
annotationProcessor "android.arch.persistence.room:compiler:1.1.1"
```

