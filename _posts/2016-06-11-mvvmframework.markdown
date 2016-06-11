---
layout:     post
title:      MVVM模式架构设计
subtitle:   databinding 和 retrofit 配合使用搭建快速框架
date:       2016-06-11
header-img: "img/post-bg-android.jpg"
tags:
- Android
- MVVM模式
--- 

###环境

AndroidStudio 2.1

###MVVM模式

![MVVM模式](http://7xnh65.com1.z0.glb.clouddn.com/2012112821430927.jpg)

MVVM模式：View，ViewModel，Model，三者关系如上图。
Android 的MVVM模式：View 在 大多情况下是指 Activity，也因为很多操作需要用到Context，Activity也充当 Controller 的角色，很多人喜欢把逻辑写在Activity中也是这个原因。在我眼里，MVVM模式解读应该是 Activity是View和分发器，逻辑处理等写在ViewModel 中需要分发的时候回调给Activity分发，而数据则是Model提供，大多数情况下Model是数据固定后不用改变的。这才是真正的MVVM模式。

Databinding的出现使得上面的假设成立，初试Databinding发现大大简便了开发，加上现在支持双向绑定，实在是太棒了。

###超高速搭建MVVM模式的库

目前MVVM模式的项目还不多，大多都是MVC，MVP模式，对于Android 的MVVM模式的架构应该怎么搭建，怎么使用，这方面完整的知识并不多见。在此抛砖引玉，提出一个方案。
Databinding可以跳过Activity的findViewById直接和xml中控件进行双向绑定，对于数据显示带来了大大的便利。我们大多数项目中列表占了很重要的一环，其中列表数据绑定、刷新、加载更多、点击响应、状态显示都是常用而且可以封装起来的。
而对于网络库，我使用了Retrofit ，返回同一了JSON格式：{"status":0,"msg":"提示消息","content":{}}  ，其中 content 里面数据如果是列表则是 JSONArray，非列表则是JSONObject。
两者相配合，实现了快速从网络获取数据并显示：

只需两句话就能完成 加载网络数据后绑定数据并显示
```java
    public Call<HttpResult<List<Model>>> onLoadListHttpRequest()；
    public void setItemLayout(int itemLayout)；
```

###Github
这个库的使用，demo 中已经给出了各种情况使用的例子了，请到[MVVMFramework](https://github.com/saiwu-bigkoo/MVVMFramework)

### 参考
[MasteringAndroidDataBinding](https://github.com/LyndonChin/MasteringAndroidDataBinding)
[binding-collection-adapter](https://github.com/evant/binding-collection-adapter)
[Two-way Android Data Binding](http://www.jianshu.com/p/c481d1f4e0b6)