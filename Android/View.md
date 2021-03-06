####  View的测量
Android系统中的MeasureSpec类用来测量View，MeasureSpec是一个32位的int值，其中高2位为测量的模式，低30位为测量的大小。
- View的测量模式
	- EXACTLY
		精确值模式,当将控件的layout_width属性设置为具体数值或指定为match_parent属性时,使用的便是该模式.
	- AT_MOST
		最大值模式,当控件的layout_width属性为wrap_content时,使用该模式.
	- UNSPECIFIED
		不指定大小的测量模式，通常情况下载绘制自定义View时才会使用。
- View类默认的onMeasure()方法只支持EXACTLY模式，故如果自定义控件时，不重写onMeasure()方法的话，只能使用EXACTLY模式。
- 系统最终是通过setMeasuredDimension(int measuredWidth,int measuredHeight)方法宽高设置到View。
- 自定义控件View测量模板
```java
	@Override
    protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec) {
 setMeasuredDimension(measureWidth(widthMeasureSpec),measureHeight(heightMeasureSpec));
    }

    private int measureWidth(int measureSpec) {
        int result = 0;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
        
        if(specMode == MeasureSpec.EXACTILT) {
            result = specSize;
        } else {
            result = 200;   // 默认值
            if(specMode == MeasureSpec.AT_MOST) {
                result = Math.min(result,specSize);
            }
        }
        return result;
    }
```

#### View的绘制
- 一般情况下，我们可以通过重写View类中的onDraw()方法来绘图，在onDraw()方法中需要一个参数，即Canvas对象。
- 在其他情况中，通常需要用代码去创建一个Canvas对象。
	- 传入的bitmap对象与创建的Canvas画布紧紧联系在一起，该过程称为**装载画布**。
	- 该bitmap对象用来储存所有绘制在Canvas上的像素信息。我们调用的所有Canvas.drawXXX方法都是作用在这个bitmap上。
```java
Canvas canvas = new Canvas(bitmap);
```

