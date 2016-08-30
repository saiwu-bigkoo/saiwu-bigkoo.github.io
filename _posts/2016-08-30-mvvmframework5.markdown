> 这一章是列表ViewModel的具体使用小结。

### Model
比如我们要请求一个门票列表，地址为http://www.bigkoo.com/api/list/ticket
参数page和pageSize分别代表当前页和一页多少个item，返回格式

```java
{"code":0,"msg":"获取详情成功","content":[{"productId":"1200001","spotName":"门票名称1"},{"productId":"1200002","spotName":"门票名称2"}]}
```

那么则建立一个叫Ticket的Model，属性有productId，spotName

```java
public class Ticket {
    /**
     * productId : 923010086
     * spotName : 蓝水河漂流
     */

    private String productId;
    private String spotName;

    public String getProductId() {
        return productId;
    }

    public void setProductId(String productId) {
        this.productId = productId;
    }

    public String getSpotName() {
        return spotName;
    }

    public void setSpotName(String spotName) {
        this.spotName = spotName;
    }

```

HttpService里面也加入对应接口

```java
@GET("api/list/ticket")Call<HttpResult<List<Ticket>>> getTicketList(@Query("page") int page, @Query("pageSize") int pageSize);
```

### 建立列表ViewModel

TicketListViewModel 就是具体实现的ViewModel，通过onLoadListHttpRequest指定请求的接口和返回的类型Ticket

```java
public class TicketListViewModel extends BaseRefreshRecyclerViewModel{

    public TicketListViewModel(){
        super(R.layout.item_ticketlist);
        onListRefresh();
    }
    @Override
    public Call<HttpResult<List<Ticket>>> onLoadListHttpRequest() {
        return HttpServiceGenerator.createService().getTicketList(getPage(),getPageSize());
    }

    @Override
    public void onItemClick(View v, int position, View itemView, Object item) {
    }
}
```

### 这个列表item对应的 item_ticketlist.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="viewModel"
            type="com.bigkoo.mvvmframeworkdemo.model.Ticket" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <TextView
            android:text="@{viewModel.spotName}"
            android:layout_width="match_parent"
            android:layout_height="50dp" />

    </LinearLayout>
</layout>

```

### 这个列表Activity对应的xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:fitsSystemWindows="true">

    <data>

        <variable
            name="viewModel"
            type="com.bigkoo.mvvmframeworkdemo.viewmodel.TicketListViewModel" />

    </data>

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <include layout="@layout/include_recyclerview_refresh"
            app:viewModel="@{viewModel}"/>

    </FrameLayout>

</layout>
```

然后就是Databinding绑定xml到Activity的知识了，具体请看源码例子。

列表的几个状态可以拓展上面Activity的xml，改为

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:fitsSystemWindows="true">

    <data>

        <variable
            name="viewModel"
            type="com.bigkoo.mvvmframeworkdemo.viewmodel.TicketListViewModel" />

    </data>

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <include layout="@layout/include_recyclerview_refresh"
            app:viewModel="@{viewModel}"/>

        <include layout="@layout/include_status_error"
            app:viewModel="@{viewModel}"/>

        <include layout="@layout/include_status_networkerror"
            app:viewModel="@{viewModel}"/>
    </FrameLayout>

</layout>
```

这里举一反三，服务器异常错误对应的xml，做成include供其他页面公用，其他状态的xml类似。还记得吗？ViewModel中的callback的时候已经封装了这些状态的改变，所以现在我们不用在ViewModel中加任何代码就能实现这些状态界面的显示和隐藏控制。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="viewModel"
            type="com.bigkoo.mvvmframework.viewmodel.BaseViewModel" />
        <variable
            name="emptyTips"
            type="String" />

        <import type="android.view.View" />

    </data>

    <LinearLayout
        android:clickable="true"
        app:setReload="@{viewModel}"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/white"
        android:gravity="center_horizontal"
        android:orientation="vertical"
        android:visibility="@{viewModel.statusError ? View.VISIBLE : View.GONE}">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:textSize="20dp"
            android:text="服务器异常" />

    </LinearLayout>

</layout>

```

> 一个带下拉刷新，滑动到最后一行加载更多，不同状态显示不同界面的列表就这么轻松的实现了。

### Github

[MVVMFramework](https://github.com/saiwu-bigkoo/Android-MVVMFramework)