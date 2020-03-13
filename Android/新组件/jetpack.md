Android Jetpack：Foundation、Architecture、Behavior、UI

## Architecture

![mvvm](/resource/mvvm.png)

### ViewModel

- 将数据独立分离出来管理，当Activity因为资源发生改变而被`onDestory`或`onCreate`时，ViewModel保存的数据不会丢失。

- 可与LiveData共同使用，从而实现对数据的监听。

- 可与Room共同使用，从而实现对数据的存储。

- 使用

  项目`build.gradle`

  ```groovy
  def lifecycle_version = "1.1.1"
  
  // ViewModel and LiveData
  implementation "android.arch.lifecycle:extensions:$lifecycle_version"
  
  implementation "android.arch.lifecycle:common-java8:$lifecycle_version"
  
  // optional - ReactiveStreams support for LiveData
  implementation "android.arch.lifecycle:reactivestreams:$lifecycle_version"
  
  // optional - Test helpers for LiveData
  testImplementation "android.arch.core:core-testing:$lifecycle_version"
  
  
  // use androidx
  def lifecycle_version = "2.1.0"
  
  // ViewModel and LiveData
  implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
  // alternatively - just ViewModel
  implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version"
  // For Kotlin use lifecycle-viewmodel-ktx
  // alternatively - just LiveData
  implementation "androidx.lifecycle:lifecycle-livedata:$lifecycle_version"
  // alternatively - Lifecycles only (no ViewModel or LiveData). Some UI
  //     AndroidX libraries use this lightweight import for Lifecycle
  implementation "androidx.lifecycle:lifecycle-runtime:$lifecycle_version"
  
  annotationProcessor "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
  // For Kotlin use kapt instead of annotationProcessor
  // alternately - if using Java8, use the following instead of lifecycle-compiler
  implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"
  
  // optional - ReactiveStreams support for LiveData
  implementation "androidx.lifecycle:lifecycle-reactivestreams:$lifecycle_version"
  // For Kotlin use lifecycle-reactivestreams-ktx
  
  // optional - Test helpers for LiveData
  testImplementation "androidx.arch.core:core-testing:$lifecycle_version"
  ```

  `MyViewModel.java`

  ```java
  public class MyViewModel extends ViewModel {
  
      private int number;
  
      public int getNumber() {
          return number;
      }
  
      public void setNumber(int number) {
          this.number = number;
      }
  }
  ```

  `ViewModelActivity.java`

  ```java
  public class ViewModelActivity extends AppCompatActivity {
  
      private TextView mTxtNumber;
      private Button mBtnAddOne;
      private Button mBtnAddTwo;
  
      private MyViewModel mViewModel;
  
      @Override
      protected void onCreate(@Nullable Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_viewmodel);
          mTxtNumber = (TextView) findViewById(R.id.txt_number);
          mBtnAddOne = (Button) findViewById(R.id.btn_addOne);
          mBtnAddTwo = (Button) findViewById(R.id.btn_addTwo);
          mViewModel = ViewModelProviders.of(this).get(MyViewModel.class);
          // 资源改变时，设置文本
          mTxtNumber.setText(mViewModel.getNumber());
          
          mBtnAddOne.setOnClickListener(view -> {
              mViewModel.setNumber(mViewModel.getNumber() + 1);
              mTxtNumber.setText(mViewModel.getNumber());
          });
          mBtnAddTwo.setOnClickListener(view -> {
              mViewModel.setNumber(mViewModel.getNumber() + 2);
              mTxtNumber.setText(mViewModel.getNumber());
          });
      }
  }
  ```

### ViewModel Savedstate

类似于之前的`onSaveInstanceState()`的方法保存简单数据。能解决程序非正常退出时数据的保存（被后台杀死），但不能解决程序正常结束的数据的保存。

```java
public class ScoreViewModel extends ViewModel {
    public static final String A_TEAM_SCORE = "aTeamScore";
    private SavedStateHandle mHandle;

    public ScoreViewModel(SavedStateHandle handle) {
        mHandle = handle;
    }

    public MutableLiveData<Integer> getaTeamScore() {
        // 根据文档，系统传来的 handle 不会为空，故不用判空
        if (!mHandle.contains(A_TEAM_SCORE)) {
            mHandle.set(A_TEAM_SCORE, 0);
        }
        return mHandle.getLiveData(A_TEAM_SCORE);
    }

    public void aTeamAdd(int score) {
        getaTeamScore().setValue(getaTeamScore().getValue() + score);
    }
}
```

- 此时ViewModel的创建

 ```java
mViewModel = new ViewModelProvider(this, new SavedStateViewModelFactory(getApplication(), this))
          .get(ScoreViewModel.class);
 ```

### LiveData

当底层数据改变时，自动通知界面，避免写一些`setText()`方法。

LiveData具有生命周期感知能力，除非Activity/Fragment处于活跃状态（已接收`onStart()`但尚未接收`onStop()`）才会调用`onChanged()`回调。同时，当调用了`onDestory()`方法时，LiveData会自动移除观察者。

`ViewModelWithLiveData.java`

```java
public class ViewModelWithLiveData extends ViewModel {

    private MutableLiveData<Integer> number;

    public ViewModelWithLiveData() {
        number = new MutableLiveData<>();
        number.setValue(0);
    }

    public MutableLiveData<Integer> getNumber() {
        return number;
    }

    public void addNumber() {
        number.setValue(number.getValue() + 1);
    }

    public void plusNumber() {
        number.setValue(number.getValue() - 1);
    }
}
```

`LiveDataActivity.java`

```java
public class LiveDataActivity extends AppCompatActivity {

    private TextView mTxtNumber;
    private ImageButton mBtnAddOne;
    private ImageButton mBtnPlusOne;

    ViewModelWithLiveData mViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_live_data);
        mTxtNumber = (TextView) findViewById(R.id.txt_number);
        mBtnAddOne = (ImageButton) findViewById(R.id.btn_addOne);
        mBtnPlusOne = (ImageButton) findViewById(R.id.btn_plusOne);

        mViewModel = ViewModelProviders.of(this).get(ViewModelWithLiveData.class);
        // 添加观察，当数据改变时，执行onChange方法
        // 第一个参数为 lifeOwner ，必须为有lifecycler管理功能的对象
        mViewModel.getNumber().observe(this, new Observer<Integer>() {
            @Override
            public void onChanged(@Nullable Integer integer) {
                mTxtNumber.setText(String.valueOf(integer));
            }
        });
        mBtnAddOne.setOnClickListener(view -> mViewModel.addNumber());
        mBtnPlusOne.setOnClickListener(view -> mViewModel.plusNumber());
    }
}
```

### DataBinding

- 以声明方式将数据绑定到界面。减少界面控件的声明和绑定。

- 可以提供界面和数据的双向绑定。

- 使用

  `项目的build.gradle`

  ```groovy
  defaultConfig {
         ...
          dataBinding {
              enabled true
          }
      }
  ```

  `activity_data_binding.xml`：使用`layout`包裹

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <layout
      xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      tools:context=".dataBinding.DataBindingActivity">
  
      <!-- name 变量名称可随意起-->
      <!-- type 为数据类型-->
      <data>
          <variable
              name="data"
              type="com.example.jetpack.dataBinding.MyViewModel" />
      </data>
  
      <RelativeLayout
          android:layout_width="match_parent"
          android:layout_height="match_parent">
  		<!--text替代赋值-->
          <TextView
              android:id="@+id/txt_grade"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_above="@+id/btn_add"
              android:layout_alignLeft="@+id/btn_add"
              android:layout_marginLeft="38dp"
              android:layout_marginBottom="30dp"
              android:textSize="25sp"
              android:text="@{String.valueOf(data.number)}" />
  		<!-- onClick替代点击事件-->
          <Button
              android:id="@+id/btn_add"
              android:text="点击增加"
              android:onClick="@{() -> data.add()}"
              android:layout_centerInParent="true"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"/>
      </RelativeLayout>
  </layout>
  ```

  `DataBindingActivity.java`

  ```java
  public class DataBindingActivity extends AppCompatActivity {
  
      MyViewModel mViewModel;
      ActivityDataBindingBinding mBinding;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          mViewModel = ViewModelProviders.of(this).get(MyViewModel.class);
          mBinding = DataBindingUtil.setContentView(this, R.layout.activity_data_binding);
  
          // 设置layout文件中变量的值，此出setData方法是因为xml中有data变量。
          mBinding.setData(mViewModel);
          // 实现liveData的自我监听
          mBinding.setLifecycleOwner(this);
      }
  }
  ```


