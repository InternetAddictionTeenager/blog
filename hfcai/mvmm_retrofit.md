# MVVM之Retrofit与LiveData的集成



对于一个健全的Android应用而言，网络请求部分必不可少，而且可以说是很重要，早些时候对于网络请求这块需要我们程序员所花费的精力不可说少，什么请求参数拼接、请求头、请求方法、请求内容等封装，以及数据model与请求、返回结果之间的格式化转换，一个个都代表着工作量，渐渐的网络请求框架丰富起来了，它做了我们绝大多数工作，像优化网络请求加载速度、请求缓存、注解封装请求等。类似Vollery、OkHttp请求的优化了我们的请求过程、缓存，而最让人欣慰的是Retrofit的出现,Retrofit本身并不处理请求，它做的是请求过程的封装、简化，让我们开发人员不必与过多的关注请求过程，而关注与结果本身。随着RxJava的出现，可以说是Retrofit请求框架迎来了巅峰，RxJava的链式、切换线程和Observer属性与Retrofit简直就是标配，让我们的请求更加简单。MVVM中采用的是LiveData实现的Observer特性，想着与RxJava的observer的功能重合，且ViewModle中本身就是用的LiveData处理，所以就想着直接通过Retrofit将LiveData进行封装替代Rxjava了。
![](https://gitee.com/amiko/articles/raw/master/android/structure/working-with-livedata.jpg)

- [MVVM的组成结构](./mvvm.md)
- [MVVM之DataBinding的使用](./databinding.md)
- [MVVM之LiveData的使用](./livedata.md)
- [MVVM之Retrofit与LiveData的集成](./mvvm_retrofit.md)
- [MVVM之ViewModel的简单封装](./mvvm_use.md)

### 三方地址

- [Retrofit](https://github.com/square/retrofit)
- [Android Architecture Blueprints (todo-mvvm-live-kotlin branch)](https://github.com/googlesamples/android-architecture/tree/todo-mvvm-live-kotlin/)

### 集成
1. Retrofit的封装

Retrofit采用单例模式封装，并设置公用头信息

```kotlin
/**
  *@Author hfcai http://www.enjoytoday.cn
  *  Retrofit工厂，单例
  */
class RetrofitFactory private constructor() {

    /*
        单例实现
     */
    companion object {
        val instance: RetrofitFactory by lazy { RetrofitFactory() }
    }

    private val interceptor: Interceptor
    private val retrofit: Retrofit

    //初始化
    init {
        //通用拦截
        interceptor = Interceptor { chain ->
            val request: Request = chain.request()
                .newBuilder()
                .addHeader("Content-Type", "application/json")
                .addHeader("charset", "UTF-8")
                .build()

            chain.proceed(request)
        }

        //Retrofit实例化
        retrofit = Retrofit.Builder()
            .baseUrl(Common.SERVER_ADDRESS)
            .addConverterFactory(GsonConverterFactory.create()) //设置数据格式化工具
            .addCallAdapterFactory(LiveDataCallAdapterFactory()) //是指代理请求返回处理
            .client(initClient())
            .build()
    }

    /*
        OKHttp创建
     */
    private fun initClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(initLogInterceptor())
            .addInterceptor(interceptor)
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    /*
        日志拦截器
     */
    private fun initLogInterceptor(): HttpLoggingInterceptor {
        val interceptor = HttpLoggingInterceptor()
        interceptor.level = HttpLoggingInterceptor.Level.BODY
        return interceptor
    }

    /*
        具体服务实例化
     */
    fun <T> create(service: Class<T>): T {
        return retrofit.create(service)
    }
}


```

其中，**addCallAdapterFactory(LiveDataCallAdapterFactory())** 即为LiveData的适配器。


2. 实现LiveData适配器
通过实现Retrofit的CallAdapter适配器来代理请求和处理返回信息，如下:

```kotlin

/**
 * @ClassName LiveDataCallAdapterFactory
 * @Description LiveData retrofit适配器
 * @Author hfcai http://www.enjoytoday.cn
 * @Date 2019/4/4 16:17
 * @Version 1.0
 */
class LiveDataCallAdapterFactory : CallAdapter.Factory() {



    /**
     * 实现allAdapter.Factory()的get方法用于返回我们实现的CallAdapter<*>对象
     */
    override fun get(returnType: Type?, annotations: Array<out Annotation>?,
                     retrofit: Retrofit?): CallAdapter<*>? {
        if(returnType !is ParameterizedType){
            throw IllegalArgumentException("返回值需为参数化类型")
        }
        //获取returnType的class类型
        val returnClass = CallAdapter.Factory.getRawType(returnType)
        if(returnClass != RespLiveData::class.java){
            throw IllegalArgumentException("MutableLiveData")
        }
        val type = CallAdapter.Factory.getParameterUpperBound(0, returnType)
        val a = LiveDataCallAdapter(type)
        LogUtils.e("get request data:${a.responseType()}")
        return a
    }
    /**
     * 请求适配器
     *
     * T:需要返回的数据RespLiveData<T>
     * R:
     */
    class LiveDataCallAdapter(/**val map: MutableMap<String,RespLiveData<*>>,**/var type:Type):CallAdapter<RespLiveData<*>>{

        override fun <R>adapt(call: Call<R>?): RespLiveData<*>? {
            var mutableLiveData:RespLiveData<*>? =null
            call?.let {
                LogUtils.e("开始请求数据")
                val requestUrl =call.request().url().encodedPath()
                LogUtils.e("请求url:$requestUrl")
                mutableLiveData = object : RespLiveData<R>(){
                    override fun onActive() {
                        super.onActive()
                        request(call)
                    }
                }
            }
            return mutableLiveData
        }


        override fun responseType(): Type = type

    }
}


```
如上，返回自定义的LiveDataCallAdapter方法，具体实现放在自定义的RespLiveData里处理


3. 自定义网络请求处理Live
我主要是把处理请求和响应下发放在自定义的LiveData里面做，当然这不是一定的，你也可以直接在LiveDataCallAdapter里匿名实现请求处理。如下为自定义的LiveData:

```kotlin
/***
 * 泛型T需是BaseResp的子类，对应的api也需要是BaseResp的子类
 */
open class RespLiveData<T>: MutableLiveData<T>() {

    //这个作用是业务在多线程中,业务处理的线程安全问题,确保单一线程作业
     val flag = AtomicBoolean(false)

    /**
     * 请求回调重写
     */
    fun<R> request(callHttp: Call<R>){
        if(flag.compareAndSet(false,true)){
            callHttp.enqueue(object: Callback<R> {
                override fun onFailure(call: Call<R>?, t: Throwable?) {
                    LogUtils.e("CallAdapter onFailure:${t?.message}")
                    postValue(null)

                }
                override fun onResponse(call: Call<R>?, response: Response<R>?) {
                    if (response?.isSuccessful!!) {
                        try {
                            val a = response.body() as T
                            postValue(a)
                        }catch (e: Exception){
                            e.printStackTrace()
                            postValue(null)
                        }
                    }else{
                          postValue(null)
                    }
                }
            })
        }
    }

}

```


4. 创建并使用

```kotlin
/**
 * @ClassName TestApi
 * @Description api请求创建
 * @Author hfcai
 * @Date 2019/4/4 17:20
 * @Version 1.0
 */

@TestRemoteApi(apiName = "测试模块")
interface TestApi{
    /**
     * 获取验证码
     */
    @POST("/api/")
    fun getTest(@Body req: TestReq): RespLiveData<TestResp>
}



//使用

class TestViewModel：ViewModel(){

    fun requestTest(){
      val req = TestReq(10)
      RetrofitFactory.instance.create(TestApi::Class.java)
                      .getTest(req)
                      .observeForever{
                        val resp:TestResp = it
                      }
    }

}


```
