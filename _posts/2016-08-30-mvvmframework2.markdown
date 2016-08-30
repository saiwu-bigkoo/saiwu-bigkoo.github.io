上一章讲到BaseViewModel中几个状态，而通常这几个状态都是根据网络返回情况来判断的，建立一个基类写一个通用的网络回调的话，就能把各种状态设置好，不用在每个具体实现的类里重复去设置了。

### Retrofit
网络库集成了Retrofit，我们通过继承Retrofit的Callback来实现自己的HttpServiceCallBack抽象类，在HttpServiceCallBack 里面定义onHttpSuccess，onHttpFail，onNetWorkError，onHttpComplete，在具体实现中设置对应的状态。

```java
public abstract class HttpServiceCallBack<T> implements Callback<HttpResult<T>> {
    @Override
    public void onResponse(Call<HttpResult<T>> call, Response<HttpResult<T>> response) {

        HttpResult<T> result = response.body();

        if (result == null) {
            //通常是服务器出错返回了非约定格式
            onHttpFail(response.code(),"网络错误,请稍后再试");
        } else {
            if (result.getCode() == HttpStatusConstants.RESULT_OK) {
                //正确返回约定的OK码
                onHttpSuccess(result.getContent(),result.getMsg());
            }
            else {
                //返回约定的其他类型码，可根据返回码进行相对应的操作
                onHttpFail(result.getCode(),result.getMsg());
            }
        }
        onHttpComplete();
    }

    @Override
    public void onFailure(Call<HttpResult<T>> call, Throwable t) {
        //网络异常或json解析失败
        onNetWorkError();
        onHttpComplete();
    }

    public abstract void onHttpSuccess(T resultData,String msg);
    public abstract void onHttpFail(int code, String msg);
    public abstract void onNetWorkError();
    public abstract void onHttpComplete();
}
```

上面HttpResult是接口对应统一返回格式

```json
//content里面是json格式代表详情
{"code":0,"msg":"获取详情成功","content":{"productId":"1200001","spotName":"门票名称"}}
//或者content里面是jsonArray格式代表列表
{"code":0,"msg":"获取详情成功","content":[{"productId":"1200001","spotName":"门票名称1"},{"productId":"1200002","spotName":"门票名称2"}]}
```

具体请求数据请去看Retrofit的用法，相信大部分人都已经在用Retrofit了，在这里不做太多解释。
> 通过上面自定义的HttpServiceCallBack，我们可以很轻松的去做断点查看数据，因为我们之后所有的网络请求返回结果都是通过HttpServiceCallBack。

### Github

[MVVMFramework](https://github.com/saiwu-bigkoo/Android-MVVMFramework)