# Mvvm 的简单封装使用


Mvvm的封装我采用的是kotlin + ARouter + Retrofit + MvvM 方式进行封装，实现一个app的基本开发需求，包括:分包、快速开发、网络请求、框架集合等。如下，介绍一些封装的基础组件，详细代码模本可参考我的开源工程。


- [MvvmSample](https://github.com/amikoj/MvvmSample)


### 界面实现

1. 界面实现是通过反射创建基类BaseMvvmActivity、BaseMvvmFragment，如下：

```kotlin
abstract class BaseMvvmFragment<in T: ViewDataBinding, V: ViewModel>() : BaseFragment(){

    /**
     * 泛型占位符
     */
    private  val tClass:Class<T>
    private  val vClass:Class<V>
    constructor(layoutId:Int):this()


    init {
        val type = javaClass.genericSuperclass
        val types = (type as ParameterizedType).actualTypeArguments
        tClass = types[0] as Class<T>
        vClass = types[1] as Class<V>
    }

    /**
     * 数据绑定生成类
     */
    private lateinit var viewBinding:T

    /**
     * 数据操作ViewModel
     */
    protected lateinit  var viewModel: V

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        activity?.let {
            if (it is AppCompatActivity){
                viewModel =  it.obtainViewModel(vClass).also { viewModel ->
                    observe(viewModel)
                }
            }
        }
        val method = tClass.getMethod("inflate", LayoutInflater::class.java,ViewGroup::class.java,
            Boolean::class.java)
        method.isAccessible =true
        viewBinding=  (method.invoke(null,layoutInflater,container,false) as T).apply {
            val objClass = javaClass
            try{
                val mt =   objClass.methods.last {
                    it.name.startsWith("set") && it.parameterTypes.size==1 && it.parameterTypes[0] == vClass
                }

                mt?.isAccessible = true
                mt.invoke(this,viewModel)
            }catch (e: Exception){
                e.printStackTrace()
                LogUtils.e("获取viewModel异常:${e.message}")
            }
            lifecycleOwner = viewLifecycleOwner
        }
        val view = viewBinding.root
        init(view)
        return view
    }

    /**
     *
     * 数据监听回调
     * @param v 绑定的viewModel
     */
    abstract fun observe(v:V)

    /**
     * 重写初始化UI或者data,非必要
     */
   open fun init(view:View)=Unit

}
```


2. recyclerview adapter基类


```kotlin
abstract class BaseAdapter<T,V: ViewDataBinding>(@LayoutRes val layoutResId:Int,list:List<T>?=null):
        BaseQuickAdapter<T, BaseHolder<V>>(layoutResId,list) {

    override fun convert(helper: BaseHolder<V>?, item: T) {
        observe(helper?.binding,helper,item)
    }


    abstract fun observe(v:V?, helper: BaseHolder<V>?, item: T)
}


class BaseHolder<V: ViewDataBinding>(val view: View): BaseViewHolder(view){

    var binding:V = DataBindingUtil.bind<V>(view)!!.apply{
        executePendingBindings()
    }
}
```


3. viewModle 实现基类

```kotlin
abstract class BaseViewModel<T> : ViewModel() {

    /**
     * 网络请求泛型占位符
     */
    private  val tClass:Class<T>?
    val resp: RepositoryImpl = ViewModelFactory.getInstance().repository
    init {
        val type = javaClass.genericSuperclass
        val types = (type as ParameterizedType).actualTypeArguments
        tClass = types[0] as Class<T>
    }

    /**
     * 网络请求api
     */
    val api:T? by lazy {
        tClass?.let {
            RetrofitFactory.instance.create(tClass)
        }
    }

    companion object {

        @BindingAdapter("app:imageUrl")
        @JvmStatic
        fun loadImage(imageView: ImageView, url: String) {
            Glide.with(imageView.context).load(url).into(imageView)
        }

    }

}
```


3. 网络请求使用
网络请求可直接创建retrofit 的api接口类，直接在viewModel调用请求

```kotlin
interface TestApi {

   @GET("/test")
   fun getTest():CallAdapterLiveData<BaseResp<TestData>>

}


//ViewModel
class TestViewModel : BaseViewModel<TestApi>(){

  fun initData(){
    lifecycleObserver?.apply {
           api?.getTest()?.observe(this, Observer {

               it.success {
                   //成功返回数据
               }.error {  
                  //返回数据异常
               }
           })

       }
  }
}

```


- [MVVM的组成结构](./mvvm.md)
- [MVVM之DataBinding的使用](./databinding.md)
- [MVVM之LiveData的使用](./livedata.md)
- [MVVM之Retrofit与LiveData的集成](./mvvm_retrofit.md)
- [MVVM之ViewModel的简单封装](./mvvm_use.md)
