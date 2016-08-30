列表说完了，接下来就是详情页了，详情页无非就是加载数据，其实很简单，和列表实现相同的规范。

### BaseDetailViewModel
原理和BaseListViewModel一样，比列表更简单，仅仅需要实现onLoadDatailHttpRequest函数即可。

```java
public class TicketDetailViewModel extends BaseDetailViewModel{
    public TicketDetailViewModel(){
        onLoad();
    }

    @Override
    public Call<HttpResult<Ticket>> onLoadDatailHttpRequest() {
        return HttpServiceGenerator.createService().getTicketDetail("10002");
    }
}
```

对应Activity的xml

```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:fitsSystemWindows="true">

    <data>

        <variable
            name="viewModel"
            type="com.bigkoo.mvvmframeworkdemo.viewmodel.TicketDetailViewModel" />

        <import type="com.bigkoo.mvvmframeworkdemo.model.Ticket"/>
    </data>


    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:text="@{((Ticket)viewModel.detail).spotName}" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:text="@{@string/productPrice(((Ticket)viewModel.detail).price??@string/zero)}" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:text="@{((Ticket)viewModel.detail).detailInfo}" />
    </LinearLayout>
</layout>
```

同样也支持 设置状态，include 对应状态的xml 进去 Activity的xml即可。

### BaseRefreshDetailViewModel
BaseRefreshDetailViewModel 则是 满足下拉刷新需求的DetailViewModel，在Activity 自行加入 SwipeRefreshLayout，给SwipeRefreshLayout添加属性：

```xml
app:onRefreshListener="@{viewModel.onRefreshListener}"
app:refreshing="@={viewModel.refreshing}
```

### Github

[MVVMFramework](https://github.com/saiwu-bigkoo/Android-MVVMFramework)