# MVVM之LiveData的使用



LiveDatas作为MVVM的一份子，很容易人健忘，不是很重要，但又不得不考虑，LiveData就个人而言其更类似一种简化的Rxjava的感觉，两者都是使用的观察者模式来实现的，相较于Rxjava而言，LiveData所能做的工作很有限，RxJava之与liveData类似与万精油，LiveData之于Rxjava又好像于一个界面数据处理专家，一个专精、一个大广(当然也不是说Rxjava不精，至少在线程切换、链式请求方面基本无出其右者)，在StackOverflow里也曾有人对此产生过讨论，大家也可围观下，地址如下:[Livedata vs Rxjava](RxJava is neither better nor worse to LiveData, it is different. Android Architecture Components was designed to give a model architectural pattern to android developers. So, if you're comfortable with LiveData, use that, or if you're comfortable with RxJava, use that. You can do all operations using both libraries. Though RxJava do contain a lot of syntactic sugar, same can be achieved using Livedata also.)。在这里，我们不做过多讨论，诚如，上面有人说的一样:**RxJava对LiveData来说既不好也不差，它是不同的。 Android架构组件旨在为Android开发人员提供模型架构模式。 因此，如果您对LiveData感到满意，请使用它，或者如果您对RxJava感到满意，请使用它。 您可以使用两个库执行所有操作。 虽然RxJava确实含有大量的语法糖，但使用Livedata也可以实现同样的效果。** 所以一句话看需求需要，反正我是用来LiveData就把RxJava替换了，毕竟后面还有一篇LiveData和Retrofit集成的文章(￣_,￣ )。


![LiveData](https://gitee.com/amiko/articles/raw/master/android/structure/live_data_android.png)


- [MVVM的组成结构](./mvvm.md)
- [MVVM之DataBinding的使用](./databinding.md)
- [MVVM之LiveData的使用](./livedata.md)
- [MVVM之Retrofit与LiveData的集成](./mvvm_retrofit.md)
- [MVVM之ViewModel的简单封装](./mvvm_use.md)



### LiveData的使用
LiveData的封装很小，就几个类，我们比较常用的主要是: **LiveData**、 **MutableLiveData**，还有用的比较少的: **ComputableLiveData** 、 **MediatorLiveData**等，这里只是介绍MutableLiveData的使用，其他有时间后续或会进行分析介绍.

LiveData的使用很简单,这里介绍两种使用方式:一种是基础使用MutableLiveData进行数据绑定、另一种是自定义一个LiveData并实现监听

#### 数据绑定

```xml


<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <import type="android.view.View"/>

        <variable
            name="viewmodel"
            type="com.example.android.architecture.blueprints.todoapp.addedittask.AddEditTaskViewModel"/>
    </data>

    <com.example.android.architecture.blueprints.todoapp.ScrollChildSwipeRefreshLayout
        android:id="@+id/refresh_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:enabled="@{viewmodel.dataLoading}"
        app:refreshing="@{viewmodel.dataLoading}">

        <ScrollView
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:paddingBottom="@dimen/activity_vertical_margin"
                android:paddingLeft="@dimen/activity_horizontal_margin"
                android:paddingRight="@dimen/activity_horizontal_margin"
                android:paddingTop="@dimen/activity_vertical_margin"
                android:visibility="@{viewmodel.dataLoading ? View.GONE : View.VISIBLE}">

                <EditText
                    android:id="@+id/add_task_title"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:hint="@string/title_hint"
                    android:maxLines="1"
                    android:imeOptions="flagNoExtractUi"
                    android:text="@={viewmodel.title}"
                    android:textAppearance="@style/TextAppearance.AppCompat.Title"/>

                <EditText
                    android:id="@+id/add_task_description"
                    android:layout_width="match_parent"
                    android:layout_height="350dp"
                    android:imeOptions="flagNoExtractUi"
                    android:gravity="top"
                    android:hint="@string/description_hint"
                    android:text="@={viewmodel.description}"/>

            </LinearLayout>
        </ScrollView>
    </com.example.android.architecture.blueprints.todoapp.ScrollChildSwipeRefreshLayout>
</layout>


```








```kotlin

class AddEditTaskViewModel(
    private val tasksRepository: TasksRepository
) : ViewModel(), TasksDataSource.GetTaskCallback {

    // Two-way databinding, exposing MutableLiveData
    val title = MutableLiveData<String>()

    // Two-way databinding, exposing MutableLiveData
    val description = MutableLiveData<String>()

    private val _dataLoading = MutableLiveData<Boolean>()
    val dataLoading: LiveData<Boolean>
        get() =_dataLoading


    private val _snackbarText = MutableLiveData<Event<Int>>()
    val snackbarMessage: LiveData<Event<Int>>
        get() = _snackbarText

    private val _taskUpdated = MutableLiveData<Event<Unit>>()
    val taskUpdatedEvent: LiveData<Event<Unit>>
        get() = _taskUpdated

    private var taskId: String? = null

    private var isNewTask: Boolean = false

    private var isDataLoaded = false

    private var taskCompleted = false

    fun start(taskId: String?) {
        _dataLoading.value?.let { isLoading ->
            // Already loading, ignore.
            if (isLoading) return
        }
        this.taskId = taskId
        if (taskId == null) {
            // No need to populate, it's a new task
            isNewTask = true
            return
        }
        if (isDataLoaded) {
            // No need to populate, already have data.
            return
        }
        isNewTask = false
        _dataLoading.value = true

        tasksRepository.getTask(taskId, this)
    }

    override fun onTaskLoaded(task: Task) {
        title.value = task.title
        description.value = task.description
        taskCompleted = task.isCompleted
        _dataLoading.value = false
        isDataLoaded = true
    }

    override fun onDataNotAvailable() {
        _dataLoading.value = false
    }


    internal fun saveTask() {
        val currentTitle = title.value
        val currentDescription = description.value

        if (currentTitle == null || currentDescription == null) {
            _snackbarText.value =  Event(R.string.empty_task_message)
            return
        }
        if (Task(currentTitle, currentDescription ?: "").isEmpty) {
            _snackbarText.value =  Event(R.string.empty_task_message)
            return
        }

        val currentTaskId = taskId
        if (isNewTask || currentTaskId == null) {
            createTask(Task(currentTitle, currentDescription))
        } else {
            val task = Task(currentTitle, currentDescription, currentTaskId)
                .apply { isCompleted = taskCompleted }
            updateTask(task)
        }
    }

    private fun createTask(newTask: Task) {
        tasksRepository.saveTask(newTask)
        _taskUpdated.value = Event(Unit)
    }

    private fun updateTask(task: Task) {
        if (isNewTask) {
            throw RuntimeException("updateTask() was called but task is new.")
        }
        tasksRepository.saveTask(task)
        _taskUpdated.value = Event(Unit)
    }
}


//在 Activity或者fragment中调用

class AddEditTaskFragment : Fragment() {


  override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
             savedInstanceState: Bundle?): View? {
         val root = inflater.inflate(R.layout.addtask_frag, container, false)
         viewDataBinding = AddtaskFragBinding.bind(root).apply {
             viewmodel = (activity as AddEditTaskActivity).obtainViewModel()

         }
         viewDataBinding.setLifecycleOwner(this.viewLifecycleOwner)

         viewDataBinding.viewmodel.taskUpdatedEvent.observe(this@AddEditTaskFragment.viewLifecycleOwner, Observer {
              // 监听回调
           })
         setHasOptionsMenu(true)
         retainInstance = false
         return viewDataBinding.root
     }

}


```

如上，解析步骤为:
- viewmodel中创建LiveData对象  
可以通过MutableLiveData来创建对象，初始情况下,LiveData包裹的value值为空

```kotlin
// 双向绑定，开发title的访问权限
val title = MutableLiveData<String>()

// 双向绑定，开发description的访问权限
val description = MutableLiveData<String>()

private val _dataLoading = MutableLiveData<Boolean>()

// 读取权限
val dataLoading: LiveData<Boolean>
    get() =_dataLoading

private val _taskUpdated = MutableLiveData<Event<Unit>>()
val taskUpdatedEvent: LiveData<Event<Unit>>
        get() = _taskUpdated


```

- 操作LiveData对象
可以手动操作修改LiveData数值，如数据库操作获取、网络请求等，也可以绑定xml通过手动输入同样达到修改的目的。

```kotlin
//同步操作
_taskUpdated.setValue(Event(Unit))

//异步操作
_taskUpdated.postValue(Event(Unit))

```

- 监听LiveData对象
完成如上步骤的话就可以对LiveData里的value改变状态进行监听了，当然这有个前提在LifecycleOwner周期内，当然我们也可以通过**observeForever**进行长期监听，但这最好在不需要的时候通过人工使用***removeObserver*进行释放.

```kotlin
//周期内监听
_taskUpdated.observe(this, Observer {
     // 监听回调
  })

//一直监听，不限周期
  _taskUpdated.observeForever(this, Observer {
       // 监听回调
    })

//释放observer
  _taskUpdated.removeObserver(observer)

```

#### 自定义网络连接LiveData

我们可以通过自定义网络链接LiveData，然后通过监听该对象实现周期内的网络检测处理，如下:

```kotlin
/**
 * @ClassName NetStateLiveData
 * @Description 网络状态监听,通过监听NetStateLiveData监听网络状态改变
 * @Author hfcai http://www.enjoytoday.cn
 * @Date 2019/4/4 9:56
 * @Version 1.0
 */
class NetStateLiveData(val context: Context) :LiveData<Boolean>(){

    private val broadcastReceiver: BroadcastReceiver = object : BroadcastReceiver() {
        override fun onReceive(p0: Context?, p1: Intent?) {
            value = NetWorkUtils.isNetWorkAvailable(context)
        }
    }

    override fun onInactive() {
        super.onInactive()
        context.unregisterReceiver(broadcastReceiver)
    }

    override fun onActive() {
        super.onActive()
        val intentFilter = IntentFilter()
        intentFilter.addAction(ConnectivityManager.CONNECTIVITY_ACTION)
        context.registerReceiver(broadcastReceiver, intentFilter)
    }



    fun refresh(){


    }
}


class NetActivity:AppCompatActivity() {

    lateinit var netLiveData:NetStateLiveData


    override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       netLiveData = NetStateLiveData(this)
       netLiveData..oserve(this, Observer {
            if (it) {
                LogUtils.e("网络访问正常")
            }else{
                LogUtils.e("网络访问异常")
            }
        })
   }


}

```


LiveData在使用过程还有几个问题需要注意的，如下列出:

- LiveData初始化不等于其内数据初始化
LiveData包裹的数据初始状态默认为null

- LiveData的观察者
LiveData 需要有观察者观察时才会调用onActive、onInactive

- LiveData的onChange
LiveData的value为一个对象时，当我们修改其中的属性时观察者并不能监听到修改，需要我们主动触发使用setValue或者postValue方法通知LiveData,如下可以发现其内通过version进行版本控制.

```java
@MainThread
protected void setValue(T value) {
   assertMainThread("setValue");
   mVersion++;
   mData = value;
   dispatchingValue(null);
}

```
