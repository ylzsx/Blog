####  单选条目对话框
```java
public void go1(View view) {
    AlertDialog.Builder builder = new AlertDialog.Builder(this);
    // 第一个参数为一个字符串数组，第二个参数为默认被选中的条目，-1为都不选中
    builder.setSingleChoiceItems(strs, -1, new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialogInterface, int i) {
            Toast.makeText(MainActivity.this, "i:" + i, Toast.LENGTH_SHORT).show();
            // dismiss()和cancel()都能关闭对话框
            dialogInterface.dismiss();
        }
    });
    builder.create().show();
}
```

#### 时间日期对话框
```java
// 获取当前时间
Date date = new Date();
Calendar calendar = Calendar.getInstance();
nowYear = calendar.get(Calendar.YEAR);
nowMonth = calendar.get(Calendar.MONTH);
nowDay = calendar.get(Calendar.DAY_OF_MONTH);
```

```java
public void go3(View view) {
    // 第二个参数为一个选择后的一个回调函数，后三个参数为默认显示日期
    DatePickerDialog dpd = new DatePickerDialog(this, new DatePickerDialog.OnDateSetListener() {
        @Override
        public void onDateSet(DatePicker datePicker, int i, int i1, int i2) {
            mBtnData.setText(i+"年 "+(i1+1)+"月 "+i2+"日");
            nowYear = i;
            nowMonth = i1;
            nowDay = i2;
        }
    }, nowYear, nowMonth, nowDay);
    dpd.show();
 }
```

#### 自定义对话框
```java
public class MyDialog extends Dialog {

    private Button mButton;

    // 父类中没有无参构造器,故必须实现
    public MyDialog(@NonNull Context context) {
        super(context);

        // 设置没有标题栏,必须放在setContentView之前
        this.requestWindowFeature(Window.FEATURE_NO_TITLE);

        this.setContentView(R.layout.dig);

        mButton = (Button) this.findViewById(R.id.btn_dig);
        mButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                MyDialog.this.dismiss();
            }
        });
    }
}
```

```java
public void go5(View view) {
    MyDialog myDialog = new MyDialog(this);
    myDialog.show();
}
```