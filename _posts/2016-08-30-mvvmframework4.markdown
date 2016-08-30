获取列表数据并显示已经通过BaseRecyclerViewModel实现了，但是列表还不支持下拉刷新功能，所以我们还必须通过Databinding双向绑定来把ViewModel的refreshing和xml的SwipeRefreshLayout控件进行绑定。

### BaseRefreshRecyclerViewModel

BaseRefreshRecyclerViewModel 其实 只需要实现OnRefreshListener 并且在onRefresh里面调用onListRefresh函数即可。
然后通过BindingConfig进行双向绑定桥接

```java
    @InverseBindingAdapter(attribute = "refreshing",event = "refreshingAttrChanged")
    public static boolean isRefreshing(SwipeRefreshLayout view) {
        return view.isRefreshing();
    }


    @BindingAdapter("refreshing")
    public static void setRefreshing(SwipeRefreshLayout view, boolean refreshing) {
        if (refreshing != view.isRefreshing()) {
            view.setRefreshing(refreshing);
        }
    }
    @BindingAdapter(value = {"onRefreshListener", "refreshingAttrChanged"}, requireAll = false)
    public static void setOnRefreshListener(final SwipeRefreshLayout view,
                                            final SwipeRefreshLayout.OnRefreshListener listener,
                                            final InverseBindingListener refreshingAttrChanged) {

        SwipeRefreshLayout.OnRefreshListener newValue = new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                if (listener != null) {
                    if (refreshingAttrChanged != null) {
                        refreshingAttrChanged.onChange();
                    }
                    listener.onRefresh();
                }
            }
        };

        SwipeRefreshLayout.OnRefreshListener oldValue = ListenerUtil.trackListener(view, newValue, R.id.onRefreshListener);
        if (oldValue != null) {
            view.setOnRefreshListener(null);
        }
        view.setOnRefreshListener(newValue);
    }

```

然后跟上一章一样，写一个通过的layout 用于 include ，include_recyclerview_refresh.xml 如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="viewModel"
            type="com.bigkoo.mvvmframework.viewmodel.BaseRefreshRecyclerViewModel" />
        <!--例子：app:adapterClassName='@{"com.bigkoo.adapter.xxxAdapter"}'-->
        <variable
            name="adapterClassName"
            type="String"/>
        <import type="me.tatarka.bindingcollectionadapter.LayoutManagers" />
    </data>


    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/swipeRefreshLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        app:onRefreshListener="@{viewModel.onRefreshListener}"
        app:refreshing="@={viewModel.refreshing}">

        <android.support.v7.widget.RecyclerView
            android:id="@+id/recyclerView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_weight="1"
            app:addOnItemClick="@{viewModel.onItemClickListener}"
            app:addOnScrollListener="@{viewModel.onScrollListener}"
            app:itemView="@{viewModel.itemViews}"
            app:items="@{viewModel.items}"
            app:addItemDecoration="@{viewModel.itemDecoration}"
            app:adapter='@{adapterClassName??"me.tatarka.bindingcollectionadapter.BindingRecyclerViewAdapter"}'
            app:layoutManager="@{viewModel.layoutManager}" />
    </android.support.v4.widget.SwipeRefreshLayout>

</layout>
```

使用起来和上一章一样，在具体的xml中include include_recyclerview_refresh.xml，然后ViewModel继承BaseRefreshRecyclerViewModel，即可下拉刷新。

### Github

[MVVMFramework](https://github.com/saiwu-bigkoo/Android-MVVMFramework)