# Android MVVM组成结构

渐渐的程序员变懒了，然后一个个框架，一个个插件应运而生，然后让程序员在越来越懒的道路上永不停止,美其名曰：**减少我们对于细节的不必要关注，而将更大的经历关注在业务层次，提高开发速度**。而这种说法得到了绝大数人的赞同，结果就是：我在这边开发框架使用的分享。好坏暂且不论，就开发效率和协同开发方面来说的确是利器，让我们较少的关注结构和协同方面，为公司节省不少时间，也不失为居家旅行的一道良方。闲话少叙，本篇主要介绍Google推出的 **Android Architecture Component** 中的 **MVVM-LiveData-kotlin** 。若说的不对的地方，还望各位看官指出纠正。


### 介绍  
MVVM已经出来了较长一段时间了，而这个模式则是将 **MVVM**、**kotlin** 两个结合而形成，显得十分合适，kotlin本身的简短、lambda写法配合LiveData的观察者模式使得代码的结构和层次更加鲜明。MVVM-LiveData_kotlin这一结构包含了多个部分内容，如下就该结构各个部分进行简要的分析、并将Retrofit与LiveData进行配对实现网络请求(替代RxJava+Retrofit).

   - [MVVM的组成结构](./mvvm.md)
   - [MVVM之DataBinding的使用](./databinding.md)
   - [MVVM之LiveData的使用](./livedata.md)
   - [MVVM之Retrofit与LiveData的集成](./mvvm_retrofit.md)
   - [MVVM之ViewModel的简单封装]()

本篇不免落入俗套的介绍下MVVM的背景介绍之中。

### MVVM 结构
首先谈谈什么是mvvm，android开发中有关于界面和数据的绑定这块一直都是研究的重点。随着APP的界面越来越多样、业务变得越来越复杂，一个界面上所需要处理的数据也就越来越多，这时候传统的findView方式显然使得开发者产生不满，所以有butterknife这样的注解绑定控件的出现，而业务与界面状态的不断变化处理渐渐冗长，就导致了Databinding这类数据绑定的出现，而MVVM就是基于databinding这类绑定式的APP开发框架。其结构如下:

![mvvm框架](http://www.enjoytoday.cn/wp-content/uploads/2019/04/mvvm.jpeg)

### Architecture Component
MVVM 的使用是Google 推出的 **[Android Architecture Component](https://github.com/googlesamples/android-architecture-components)** 实现，其中所包含的组件如下:

- **ROOM**  
Room 是google 对于本地数据库的一个封装，通过注解实现的一个本地数据库的创建管理组件，如下为一个Room使用的案例：

```kotlin
@Database(entities = arrayOf(Task::class), version = 1)
abstract class ToDoDatabase : RoomDatabase() {

    abstract fun taskDao(): TasksDao

    companion object {

        private var INSTANCE: ToDoDatabase? = null

        private val lock = Any()

        fun getInstance(context: Context): ToDoDatabase {
            synchronized(lock) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(context.applicationContext,
                            ToDoDatabase::class.java, "Tasks.db")
                            .build()
                }
                return INSTANCE!!
            }
        }
    }

}

```

- **Lifecycle-aware components**  
Lifecycle-aware是Google用于管理生命周期的一个组件.用于处理MVVM中数据绑定的状态变化，让我们不需要关心View的状态改变，MVVM 中的View层 **FragmentActivity、androidx.fragment.app.Fragment**  组件实现了该组件。

- **ViewModels**  
MVVM框架的VM层，用于实现与view的双向绑定，并操控model的一个组件.viewmodel主要是用于将MVVM的界面与逻辑进行解耦分离，便于代码的后期维护，其实现如下:

```kotlin

class BaseViewModel : ViewModel() {

 // viewmodel实现层

}

```
- **LiveData**  
Google开发的一个观察者模式的组件，可以替代RxJava的部分功能，是一个类似RxJava的组件。

- **Databinding**  
Databinding 是用于界面与数据的绑定使用的，先已在Andorid Studio内置，可直接通过在build.gradle设置如下代码开启:

```gradle

android {

  ...
  dataBinding {
    enabled = true
 }
}
```
