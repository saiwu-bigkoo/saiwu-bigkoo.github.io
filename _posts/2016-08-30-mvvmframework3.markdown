统一规范的BaseViewModel和HttpServiceCallBack已经建好，那么把两者关联起来实现加载列表内容的时刻到了。
列表特性就是分页去加载数据，默认的按 page（当前第几页） 和 pageSize（一页多少个item）来控制分页，而当加载数据返回的item 数量比 pageSize 小则视为没有更多数据了，所以列表拓展BaseViewModel多一个hasMore的状态判断是否有下一页数据。
因为列表有个下拉刷新的概念，加载第一页的时候认为是刷新中状态。

然后我们在BaseListViewModel中实现上面提到的逻辑，并且实现HttpServiceCallBack来设置对应状态。
```java
    public HttpServiceCallBack callBack = new HttpServiceCallBack<List<T>>() {

        @Override
        public void onHttpSuccess(List<T> resultData, String msg) {
            setStatusError(false);
            setStatusNetworkError(false);

            if(isFirstPage()) {
                items.clear();
            }
            if(resultData != null) {
                items.addAll(items.size() - footers.size(),resultData);
                //如果获取的数据数量比申请的数量少 则为没有更多了
                hasMore.set(resultData.size() < pageSize ? false : true);
            }
        }

        @Override
        public void onHttpFail(int code, String msg) {
            setStatusError(true);
        }

        @Override
        public void onNetWorkError() {
            setStatusNetworkError(true);
        }

        @Override
        public void onHttpComplete() {
            once = true;
            setStatusLoading(false);
            if(!getStatusError().get()&&!getStatusNetworkError().get())
                setStatusEmpty(items.isEmpty());

            if(isFirstPage())//因为在刷新之前已经把page设为了firstPage，所以可以判断isFirstPage()来判断当前是否刷新
                setRefreshing(false);
            else
                loadingMore.set(false);

            onLoadListComplete();
        }
    };
```

callBack中已经进行了各种状态的设置，根据Databinding特性，只要在xml中绑定了对应属性即可显示隐藏对应View。
然后通过提供一个onLoadListHttpRequest抽象函数，只要继承BaseListViewModel的子类实现onLoadListHttpRequest函数即可轻松关联请求的接口。而onLoadListComplete提供出来

然后继续拓展Header和Footer，具体请看源码。

items通过Databinding绑定layout，从构造函数中传入layout 的id，通过ItemViewSelector来进行绑定，具体原理请参考[binding-collection-adapter](https://github.com/evant/binding-collection-adapter)

因为现在都用RecyclerView了，ListView控件我早已弃用，基于BaseListViewModel 再拓展 RecyclerView专属的BaseRecyclerViewModel，主要实现setItemDecoration，setLayoutManager，onItemClickListener，onScrollListener。然后在BindingConfig里面编写转换器：

```java
    @BindingAdapter({"addOnItemClick"})
    public static void addOnItemClick(RecyclerView view, RecyclerViewItemClickSupport.OnItemClickListener listener) {
        RecyclerViewItemClickSupport.addTo(view).setOnItemClickListener(listener);
    }

    @BindingAdapter({"addOnScrollListener"})
    public static void addOnScrollListener(RecyclerView view, RecyclerView.OnScrollListener listener) {
        if(listener!=null)
            view.setOnScrollListener(listener);
    }

    @BindingAdapter({"addItemDecoration"})
    public static void addItemDecoration(RecyclerView view, RecyclerView.ItemDecoration itemDecoration) {
        if(itemDecoration != null)
            view.addItemDecoration(itemDecoration);
    }
```

那么对应在xml中加入属性：
```xml
        app:addOnItemClick="@{viewModel.onItemClickListener}"
        app:addOnScrollListener="@{viewModel.onScrollListener}"
        app:addItemDecoration="@{viewModel.itemDecoration}"
```

这样就完成绑定。那么我们把这个xml写成通用的，并加入layout_behavior兼容CoordinatorLayout进行toolbar等联动效果，把这些变为一个include文件即可。完整的xml如下（include_recyclerview.xml）：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="viewModel"
            type="com.bigkoo.mvvmframework.viewmodel.BaseRecyclerViewModel" />

        <variable
            name="adapterClassName"
            type="String" />

        <import type="me.tatarka.bindingcollectionadapter.LayoutManagers" />
    </data>


    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_weight="1"
        app:adapter='@{adapterClassName??"me.tatarka.bindingcollectionadapter.BindingRecyclerViewAdapter"}'
        app:addOnItemClick="@{viewModel.onItemClickListener}"
        app:addOnScrollListener="@{viewModel.onScrollListener}"
        app:itemView="@{viewModel.itemViews}"
        app:addItemDecoration="@{viewModel.itemDecoration}"
        app:items="@{viewModel.items}"
        app:layoutManager="@{viewModel.layoutManager}"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />
</layout>
```

如此，通用的RecyclerViewModel完成。使用起来就相当方便了。
新建一个ViewModel extends BaseRecyclerViewModel，重写onLoadListHttpRequest，onItemClick函数，在构造函数中把列表的 item layout xml通过super传给父类。然后在xml 中 include 上面的 include_recyclerview.xml：

```xml
<include layout="@layout/include_recyclerview_refresh"
            app:viewModel="@{viewModel}"/>
```

一切就是这么简单，ViewModel 里面 只要传入item layout，告诉ViewModel请求什么地址，点击做什么操作，onListRefresh之后一个列表就呈现在你眼前。

### 玩出花样

ViewModel里面设置

- setItemDecoration  设置RecyclerView的ItemDecoration。
- setLayoutManager  设置RecyclerView的样式，linear，grid，staggeredGrid。
- setSpecialView  设置RecyclerView的item 特别样式
- addHeader  设置RecyclerView的 headerView
- addFooter  设置RecyclerView的 footerView

### Github

[MVVMFramework](https://github.com/saiwu-bigkoo/Android-MVVMFramework)