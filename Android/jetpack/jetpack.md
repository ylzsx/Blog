Android Jetpack：Foundation、Architecture、Behavior、UI

## Architecture

### ViewModel

- 将数据独立分离出来管理，当Activity因为资源发生改变而被`onDestory`或`onCreate`时，ViewModel保存的数据不会丢失。

- 可与LiveData共同使用，从而实现对数据的监听。

- 可与Room共同使用，从而实现对数据的存储。

- 使用

  项目`build.gradle`

  ```
  def lifecycle_version = "1.1.1"
  
  // ViewModel and LiveData
  implementation "android.arch.lifecycle:extensions:$lifecycle_version"
  
  implementation "android.arch.lifecycle:common-java8:$lifecycle_version"
  
  // optional - ReactiveStreams support for LiveData
  implementation "android.arch.lifecycle:reactivestreams:$lifecycle_version"
  
  // optional - Test helpers for LiveData
  testImplementation "android.arch.core:core-testing:$lifecycle_version"
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