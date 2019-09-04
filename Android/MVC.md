####  MVC
1. 内存泄露
Activty中涉及到线程，网络时，很难控制运行时间，容易出现内存泄露。
控件传参到线程中很容易出现内存泄露。

#### MVP
```java
view层
interface IGirlView {
	// 不能这么写，会引起等待时间
    // List showGirlView();
    void showGirlView(List<Girl> girls);
}

interface IGirlModel {
	// 也会有等待
    // void loadGirlData(List<Girl> data);
    void loadGirlData(OnLoadListener onLoadListener);
    interface OnLoadListener {
        void onComplete(List<Girl> girls);
    }
}

public class GirlPresenter<T extends IGirlView> {
    // 需要持有view的引用
    //IGirlView iGirlView;
    WeakReference iGirlView;
    // 需要持有model引用
    IGirlModel iGirlModel = new GirlModel();
    public GirlPresenter(T view) {
        iGirlView = new WeakReference(view);
    }
    
    public void fetch() {
        if(iGirlView!=null && iGirlModel!=null) {
        iGirlModel.loadGirlData(new IGirlModel.OnloadListener() {
        		@Override
                public void onComplete(List<Girl> girls) {
                	iGirlView.showGirlView(girls);
                }
       		});
        }
    }
}
```