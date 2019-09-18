# MVVM 中的Databinding


Android 中的MVVM模式的实现其很大一部分依托于Android Architecture Component 中的Databinding的实现,DataBinding让我们的数据和界面产生了连接，而不需要我们手动的操作着令人烦闷的控件赋值操作。MVVM也正是借助于DataBinding实现了数据与界面的解耦。其类似与一种轻量级的标记语言，通过ide的支持，实现界面支持基础的标记语法操作，并通过编译布局文件将对应的语法转换为多个与之绑定的实现类来负责Model与数据之间的绑定，由此可见其发展方向类似与前端，知道前端的同学应该会发现这很类似与java中的jsp、python的模板语言，但就目前而言，其功能相对较弱，并不能如前面两种标记语言那般强大到可以直接实现布局与代码的混合开发，但未来可期。

![databinding](https://gitee.com/amiko/articles/raw/master/android/structure/binding.png)


- [MVVM的组成结构](./mvvm.md)
- [MVVM之DataBinding的使用](./databinding.md)
- [MVVM之LiveData的使用](./livedata.md)
- [MVVM之Retrofit与LiveData的集成](./mvvm_retrofit.md)
- [MVVM之ViewModel的简单封装]()


## DataBinding
DataBinding的使用Google给了不少的示例，写的也挺全面，除了都是英文外没什么缺点，基本上DataBinding的各种使用方式都有对应的操作，相较于网上充斥的介绍文档，个人比较推荐看google提供的demo案例，看的仔细了你会有一种 **麻雀虽小、五脏俱全** 的感觉,让你感觉Google大爷还是你家大爷。这里给出了Goolge提供相关的DataBinding操作的示例导航以及使用的简要介绍.

#### DataBindng 案例
这里给出了Google的有关DataBiding的示例最后一个是我用的MVVM模式的案例(MVVM集数成了DataBinding),DataBidng单独使用的话其优势并不是很大，一般而言DataBinding都是配合着观察者使用的，如下的案例大多都是使用Observer数据或者LiveData数据配合使用。

- [DataBindingBasicSample](https://github.com/googlesamples/android-databinding/blob/master/BasicSample)  
- [DataBindingTwoWaySample](https://github.com/googlesamples/android-databinding/blob/master/TwoWaySample)
- [Android Architecture Blueprints (todo-mvvm-live-kotlin branch)](https://github.com/googlesamples/android-architecture/tree/todo-mvvm-live-kotlin/)
- [GithubBrowserSample](https://github.com/googlesamples/android-architecture-components/tree/master/GithubBrowserSample)
- [Android Sunflower](https://github.com/googlesamples/android-sunflower)
- [MVVM案例](https://github.com/amikoj/AppFrameWork/tree/master/mvvm)


#### DataBinding的绑定
DataBidng 的绑定主要分为：变量绑定、事件绑定和适配器绑定，其绑定方式有分为单向绑定和双向绑定，其实现也都稍有不同。

##### 变量绑定

变量的绑定是通过实现传入的绑定对象，通过绑定的对象的参数进行绑定，同时也支持表达式、方法输出等,如下示例:

```xml
<lanyout  xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools>

  <data>
      <import type="com.example.android.databinding.basicsample.R"/>
      <import type="com.example.android.databinding.basicsample.util.ConverterUtil"/>
       <variable
            name="user"
            type="com.example.android.databinding.basicsample.data.ObservableFieldProfile" />
  </data>

  <androidx.constraintlayout.widget.ConstraintLayout
      android:layout_width="match_parent"
      android:layout_height="match_parent">

      <!--user对象的变量实现 -->
      <TextView
                android:id="@+id/name"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_marginEnd="128dp"
                android:layout_marginStart="16dp"
                android:layout_marginTop="8dp"
                android:text="@{user.name}"
                android:textAppearance="@style/TextAppearance.AppCompat.Large"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toBottomOf="@+id/name_label"/>


    <!--表达式实现 -->
      <ImageView  android:id="@+id/imageView"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_marginEnd="24dp"
      android:layout_marginTop="24dp"
      android:contentDescription="@string/profile_avatar_cd"
      android:minHeight="48dp"
      android:minWidth="48dp"
      app:layout_constraintTop_toBottomOf="@+id/name"
      app:layout_constraintStart_toStartOf="parent"
      android:tint="@{user.likes > 9 ? @color/star : @android:color/black}"
      app:srcCompat="@{user.likes < 4 ? R.drawable.ic_person_black_96dp : R.drawable.ic_whatshot_black_96dp }"/>

      <!--静态方法实现 -->
      <ProgressBar
          android:id="@+id/progressBar"
          style="?android:attr/progressBarStyleHorizontal"
          android:layout_width="0dp"
          android:layout_height="wrap_content"
          android:layout_marginEnd="8dp"
          android:layout_marginStart="8dp"
          android:layout_marginTop="8dp"
          android:max="@{100}"
          android:visibility="@{ConverterUtil.isZero(user.likes)}"
          app:progressScaled="@{user.likes}"
          app:layout_constraintTop_toBottomOf="@+id/imageView"
          app:layout_constraintStart_toStartOf="parent"
          tools:progressBackgroundTint="@android:color/darker_gray"/>

  </androidx.constraintlayout.widget.ConstraintLayout>

</layout>

```

##### 事件绑定

实现的绑定主要指的是layout中提供的方法传入实现(基本就是代指onClick方法,当然也可自行进行定制)，事件的绑定方式有两种：一种是lambda方式、另一种是保证和andorid实现方法参数相同的只需要传入方法名，如下:


```kotlin

/**
  *带绑定viewmodle
  */
class ProfileLiveDataViewModel : ViewModel() {
        fun onLike() {
           _likes.value = (_likes.value ?: 0) + 1
        }

        fun disLike(view:View) {
           _unlikes.value = (_unlikes.value ?: 0) + 1
        }

}

```



```xml
<lanyout  xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools>
  <data>
      <import type="com.example.android.databinding.basicsample.R"/>
      <import type="com.example.android.databinding.basicsample.util.ConverterUtil"/>
      <variable
            name="viewmodel"
            type="com.example.android.databinding.basicsample.data.ProfileLiveDataViewModel"/>
  </data>

  <androidx.constraintlayout.widget.ConstraintLayout
      android:layout_width="match_parent"
      android:layout_height="match_parent">

      <!--lambda方式实现 -->
      <Button
        android:id="@+id/like_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginTop="16dp"
        android:onClick="@{() -> viewmodel.onLike()}"
        android:text="@string/like"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintStart_toStartOf="@+id/imageView"
        app:layout_constraintTop_toBottomOf="@+id/likes"/>


       <!--原生样式实现 -->
        <Button
        android:id="@+id/unlike_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginTop="16dp"
        android:onClick="@{viewmodel.disLike}"
        android:text="@string/like"
        app:layout_constraintTop_toBottomOf="@id/like_button"
        app:layout_constraintStart_toStartOf="@+id/imageView"
        app:layout_constraintTop_toBottomOf="@+id/likes"/>


  </androidx.constraintlayout.widget.ConstraintLayout>

</layout>

```

##### 适配器绑定

说起DataBinding的适配器绑定就有点高级了，它是一个类似kotlin的扩展一样的东西，只不过kotlin的扩展作用的是实现代码的具体类中，而DataBinding的适配器作用到的是布局文件中的view的属性中，它可以帮我们减少很多麻烦的操作，让我们的代码看起来更具有可读性、美观性，如下:


```kotlin
/**
 * Databinding适配器的使用
 * 需要注意的是由于适配器使用的是java的static方法,为了适配kotlin这里每个适配器方法均需要添加注解@JvmStatic使得其可以与java适配
 * 详细请参考kotlin官网静态的使用
 *
 *
 */
object BindingAdapters {
    /**
     *
     *  一个绑定的适配器可以在任何地方陪使用用于设置ImageView的Popularity，接受值为popularity
     *
     */
    @BindingAdapter("app:popularityIcon")
    @JvmStatic fun popularityIcon(view: ImageView, popularity: Popularity) {

        val color = getAssociatedColor(popularity, view.context)

        ImageViewCompat.setImageTintList(view, ColorStateList.valueOf(color))

        view.setImageDrawable(getDrawablePopularity(popularity, view.context))
    }
  }

```


```xml
<lanyout  xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools>
  <data>
      <import type="com.example.android.databinding.basicsample.R"/>
      <import type="com.example.android.databinding.basicsample.util.ConverterUtil"/>
      <variable
            name="viewmodel"
            type="com.example.android.databinding.basicsample.data.ProfileLiveDataViewModel"/>
  </data>

  <androidx.constraintlayout.widget.ConstraintLayout
      android:layout_width="match_parent"
      android:layout_height="match_parent">

      <ImageView
            android:id="@+id/imageView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginEnd="24dp"
            android:layout_marginTop="24dp"
            android:contentDescription="@string/profile_avatar_cd"
            android:minHeight="48dp"
            android:minWidth="48dp"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:popularityIcon="@{viewmodel.popularity}"/>


  </androidx.constraintlayout.widget.ConstraintLayout>

</layout>

```

##### 双向绑定

DataBinding其本身并不支持双向绑定，一般都是通过一个Observer类型数据进行观察实现的双向绑定的过程，一般实现的方式分为两种:一种是Observabler接口的实现，另一种则是LiveData的绑定实现.这里有几个小的地方需要注意点：xml中一般使用 **@{}** 用于给控件赋值，xml中一般使用 **=@{}**实现xml控件的赋值和控件值变化修改对应变量的值。


- Observer接口的实现  

Observer接口实现原理说来也简单，但操作并不简单，它是DataBinding内部封装的一个接口，代理实现属性的变化自动更新ui界面，但代码中变量的变化需要我们主动触发通知UI界面进行更新,其实现如下:


```kotlin
/**
 *作为实现Observer功能的一个viewmodle基类
 */
open class ObservableViewModel : ViewModel(), Observable {

    private val callbacks: PropertyChangeRegistry = PropertyChangeRegistry()

    override fun addOnPropertyChangedCallback(callback: Observable.OnPropertyChangedCallback) {
        callbacks.add(callback)
    }

    override fun removeOnPropertyChangedCallback(callback: Observable.OnPropertyChangedCallback) {
        callbacks.remove(callback)
    }

    /**
     * 这个方法调用会对model下的所有属性进行检查并更新ui
     */
    fun notifyChange() {
        callbacks.notifyCallbacks(this, 0, null)
    }

    /**
     *
     *
     *  更新modle下制定id的控件值，其中id是databinding对应生成的一个变量id，不同于控件的属性id,databinding其本质是控件与属性的一一绑定.
     *
     * @param fieldId The generated BR id for the Bindable field.
     */
    fun notifyPropertyChanged(fieldId: Int) {
        callbacks.notifyCallbacks(this, fieldId, null)
    }
}




/**
  * viewmodle的实现类，具体需要绑定的viewmodle,主要对于需要双向绑定的需要手动提醒更新
  */
class ProfileObservableViewModel : ObservableViewModel() {
    val name = ObservableField("Ada")
    val lastName = ObservableField("Lovelace")
    val likes =  ObservableInt(0)

    fun onLike() {
        likes.increment()
        //需要手动提醒更新
        notifyPropertyChanged(BR.popularity)
    }

    @Bindable
    fun getPopularity(): Popularity {
        return likes.get().let {
            when {
                it > 9 -> Popularity.STAR
                it > 4 -> Popularity.POPULAR
                else -> Popularity.NORMAL
            }
        }
    }
}

enum class Popularity {
    NORMAL,
    POPULAR,
    STAR
}

private fun ObservableInt.increment() {
    set(get() + 1)
}

```


- LiveData绑定实现

LiveData的实现相对就比较容易写,LiveData其本省就是被作为一个Observer监听模式的一个存在，如下:

```kotlin

class ProfileLiveDataViewModel : ViewModel() {
    private val _name = MutableLiveData("Ada")
    private val _lastName = MutableLiveData("Lovelace")
    private val _likes =  MutableLiveData(0)

    val name: LiveData<String> = _name
    val lastName: LiveData<String> = _lastName
    val likes: LiveData<Int> = _likes

    // popularity is exposed as LiveData using a Transformation instead of a @Bindable property.
    val popularity: LiveData<Popularity> = Transformations.map(_likes) {
        when {
            it > 9 -> Popularity.STAR
            it > 4 -> Popularity.POPULAR
            else -> Popularity.NORMAL
        }
    }

    fun onLike() {
        _likes.value = (_likes.value ?: 0) + 1
    }
}
```

其上所实现的功能相同，就开发的便捷性而言个人比较推荐LiveData实现双向绑定，而且LiveData本身还有其他好玩的方式等待我们的探索。本篇介绍的代码类似与伪代码性质，不具有贯通性，具体实现的使用个人还是比较推荐大家可以去看看我上面推荐的DataBinding的demo示例代码，那里有你想要的一切，该有的它都有！另外，需要注意的是，在我们需要的model里添加设置代码:

```gradle
android {

   .....
   dataBinding {
    enabled true
   }

}
```
