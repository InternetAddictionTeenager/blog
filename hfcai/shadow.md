# Android 布局阴影实现

最近项目要求，ui有很多有关于阴影的设计要求，网上找了些实现方式，但都不是很理想。现在闲下来了，就寻思着自己写个阴影布局耍耍，以备后用。先说道说道我找到的几种阴影实现方式:


#### 系统阴影

Andorid 系统自api 21之后就多了一个熟悉 **android:elevation** ，这是android最新引入的轴的概念，可通过设置elevation来设置阴影(z轴的大小)，设置如下:

```xml
<!-- base z depth of the view. -->
  <attr name="elevation" format="dimension" />

  <TextView
        android:id="@+id/shadow1"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        android:layout_marginStart="20dp"
        android:layout_marginTop="20dp"
        android:text="系统阴影"
        android:background="#fff"
        android:gravity="center"
        android:textSize="14sp"
        android:textColor="@color/colorBlack"
        android:layout_width="100dp"
        android:elevation="3dp"
        android:layout_height="80dp"/
```

效果也是不错，可以完成一些简单的阴影设置效果。
<img src="https://i.loli.net/2019/09/17/FK6GwpQEbUnyhCZ.png" width = 30% height = 30% />

但需要注意些细节，不然 **elevation** 可能会无效：

- 父布局要保留足够的空间，elevation本身不占有view的大小
- **需要设置背景色且不可设置为透明色**
- 不能设置是否为扩散的还是指定方向的



#### layer-list 伪阴影

为什么说是伪阴影呢，layer-list本身很强大，器支持的层叠式绘制基本可以解决我们大多说的背景设计，对于一些要求不是很严格的阴影用它也不是不可以，但效果是真的不好，毕竟shape提供的层叠()并不支持模糊绘制(或者可以选择使用模糊背景图片绘制)。下面给一个用layer-list绘制的阴影做参考。


```xml
<TextView
    android:id="@+id/shadow2"
    app:layout_constraintStart_toEndOf="@id/shadow1"
    app:layout_constraintTop_toTopOf="parent"
    android:layout_marginStart="50dp"
    android:layout_marginTop="20dp"
    android:text="layer-list阴影"
    android:gravity="center"
    android:background="@drawable/shadow_layer"
    android:textSize="14sp"
    android:textColor="@color/colorBlack"
    android:layout_width="100dp"
    android:layout_height="80dp"/>



<!--shadow_layer.xml -->
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item android:top="3dp"
        android:left="3dp">
        <shape android:shape="rectangle">
            <solid android:color="#333333"/>
            <gradient android:startColor="#80ff0000"
                android:type="radial"
                android:centerX="0.5"
                android:centerY="0.5"
                android:gradientRadius="30"
                android:endColor="#10ff0000"/>
            <size android:width="100dp" android:height="80dp"/>
        </shape>
    </item>

    <item android:bottom="3dp"
        android:right="3dp">
        <shape android:shape="rectangle">
            <solid android:color="#fff"/>
            <size android:width="100dp" android:height="80dp"/>
        </shape>
    </item>

</layer-list>

```

效果比较生硬，其本质就是颜色的渐变，如下:
<img src="https://i.loli.net/2019/09/17/f27ELOdIFSWJXuc.png" width = 30% height = 30% />


还有如让ui切阴影背景图，但由于控件大小规格差异较大，风格差异较大，并不推荐使用。

### 自定义阴影布局
这是我比较推荐的方式,可参考CardView的阴影实现自定义一个阴影布局实现。其实现是通过 **setShadowLayer**、**setMaskFilter** 实现。
```java
//        mPaint.setShadowLayer(blurRadius,0,0,shadowColor);  
        if (blurRadius>0){
            mPaint.setMaskFilter(new BlurMaskFilter(blurRadius,BlurMaskFilter.Blur.NORMAL));
        }
```

相较于 **setShadowLayer** 来说，**setMaskFilter** 可供选中的实现方式要多一个blur实现类型，效果更好些，所以我是通过使用 **setMaskFilter** 来实现自定义阴影布局。


```xmlns

<cn.enjoytoday.shadow.ShadowLayout
    android:id="@+id/shadow3"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@id/shadow1"
    android:layout_marginTop="20dp"
    android:text=""
    app:shadowRadius="0dp"
    app:shadowColor="#333"
    app:blurRadius="5dp"
    app:xOffset="0dp"
    app:yOffset="0dp"
    android:layout_marginStart="15dp"
    android:gravity="center"
    android:background="@drawable/shadow_layer"
    android:textSize="14sp"
    android:textColor="@color/colorBlack"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">


    <TextView
        android:padding="5dp"
        android:text="自定义应用布局"
        android:gravity="center"
        android:textSize="14sp"
        android:textColor="@color/colorBlack"
        android:layout_width="100dp"
        android:layout_height="80dp"/>

</cn.enjoytoday.shadow.ShadowLayout>
```

<img src="https://i.loli.net/2019/09/17/AIFzgTrep9CijwK.png" width = 30% height = 30% />

### 使用
[ShadowView](https://github.com/amikoj/ShadowView) 托管于GitHub, 仿照css的Box Shadow 的阴影实现效果设计实现，可通过设置水平、竖直偏移确认阴影方向，可设置模糊半径和圆角半径、阴影颜色等。可通过gradle直接依赖使用:

#### 添加依赖

```gradle
repositories {
	//...
	maven { url 'https://jitpack.io' }
}

dependencies {
    implementation 'com.github.amikoj:ShadowView:1.0.1'
}

```


#### xml中使用


```xml

   <cn.enjoytoday.shadow.ShadowLayout
        android:orientation="vertical"
        android:id="@+id/shadowLayout"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        android:gravity="center"
        app:shadowRadius="10dp"
        app:shadowColor="#bebebe"
        app:bgColor="#fff"
        app:xOffset="10dp"
        app:yOffset="0dp"
        app:blurRadius="5dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">

   <!--嵌套需要添加阴影的布局 -->

    </cn.enjoytoday.shadow.ShadowLayout>
```


#### 属性说明



 |   属性名      | 类型    |  说明  |
 | --------   | -----:   | :----: |
 |   shadowColor      | color      |   阴影渲染颜色   |
 | shadowRadius        | dimension      |   背景圆角半径(0为矩形)    |
 | blurRadius        | dimension      |   模糊半径    |
 | xOffset        | dimension      |   水平位移  |
 | yOffset        | dimension      |   竖直位移  |
 | bgColor        | color      |     背景色  |




#### 代码设置

 也可通过代码设置阴影属性：

```java
shadowLayout.getShadowConfig()   //获取配置类
            . setBlurRadius(blurRadius)  //设置模糊半径
             .setXOffset(xoffset)   //设置水平位移，最大为20dp
             .setYOffset(yoffset)   //设置竖直位移，最大为20dp
             .setShadowRadius(shadowRadius) //设置圆角半径，为0时不是圆角
             .setShadowColor(shadowColor)    //设置阴影颜色
             .commit();             //生效修改
```
