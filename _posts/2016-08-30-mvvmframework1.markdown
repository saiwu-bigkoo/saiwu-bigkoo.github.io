MVVMFramework 是基于Databinding上建立一套MVVM代码规范，继承对应的BaseViewModel就能轻松实现快速MVVM模式开发。
Databinding资料相对还是比较少，但是官方给出已经很全面了，入门可能有点难，但是上手后开发效率可谓是极速，下面来介绍介绍这套基于Databinding建立的MVVMFramework。

> 通常开发一个APP，肯定离不开列表和详情。那么何不把这两个模块能够通用的部分抽出来呢？接下来思考：哪些都是重复的代码？Databinding能够实现到什么程度？

### BaseViewModel

每个需要加载需要网络数据的页面其实都有几个通用的状态。比如进入一个列表，首次加载就会有一个一整页的加载中提示View，有数据之后加载数据的状态则变为下拉刷新和加载更多，而获取数据有可能网络异常而获取不到，而数据拿回来之后可能解析出现错误或者服务器崩溃，或仅仅是列表数据没有内容。比如详情页，除了没有加载更多这个状态之外，其他的状态也同样是有的。
统计了上面的几种情况，通用的都有：加载中，刷新中，空数据，错误，网络异常 这几种状态。

```java
public abstract class BaseViewModel<T> {
    //刷新状态
    private final ObservableBoolean refreshing = new ObservableBoolean(false);
    //空数据状态
    private final ObservableBoolean statusEmpty = new ObservableBoolean(false);
    //加载中状态
    private final ObservableBoolean statusLoading = new ObservableBoolean(false);
    //错误状态
    private final ObservableBoolean statusError = new ObservableBoolean(false);
    //网络异常状态
    private final ObservableBoolean statusNetworkError = new ObservableBoolean(false);
    //通知View进行交互的监听器
    private OnViewModelNotifyListener onViewModelNotifyListener;

//......篇幅原因各状态的getset方法省略

    public void setOnViewModelNotifyListener(OnViewModelNotifyListener onViewModelNotifyListener) {
        this.onViewModelNotifyListener = onViewModelNotifyListener;
    }

    /**
     * 通知View进行交互
     * @param bundle 装载数据
     * @param code 判别View要做什么操作
     */
    public void onViewModelNotify(Bundle bundle, int code){
        if(onViewModelNotifyListener != null)
            onViewModelNotifyListener.onViewModelNotify(bundle,code);
    }

    /**
     * 加载数据
     */
    public void onLoad(){}
}
```

> BaseViewModel 定义了一些列通用状态和预留了Activity和ViewModel之间的回调接口，而onLoad则是加载数据时通用的方法。

那么我们只需要继承这个BaseViewModel ，然后在子类的onLoad函数里面获取网络数据，在开始获取网络和网络回调中set不同的状态，这样Databinding就会改变关联到界面上对应你设置的状态。比如在onLoad里面获取数据，回调之后数据为空，则在子类ViewModel里面设置一下setStatusEmpty(true);然后在xml里面设置一整屏大小的TextView，如果获取的数据为空的话就显示：

```xml
        <TextView
            android:visibility="@{viewModel.statusEmpty ? View.VISIBLE : View.GONE}"
            android:background="@android:color/white"
            android:gravity="center"
            android:textSize="30sp"
            android:text="没有数据。。。。。"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
```

这样各个状态需要显示和隐藏的内容都会因为ViewModel设置对应状态true或false控制显隐。


### Github

[MVVMFramework](https://github.com/saiwu-bigkoo/Android-MVVMFramework)