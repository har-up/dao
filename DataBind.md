# Data Binding

## 介绍

Data Binding是一个类库，可以让你在布局文件中声明式的绑定数据以显示。其中的数据由ViewModel提供，很好的支持MVVM架构开发。Data Binding 可以解决下面一般开发过程中存在的几个问题。

- 在不是用databind的情况下，我们会需要展示的数据赋予控件，一般需要findViewById(),不断改变值还要findViewById，而findViewById的消耗并不低且有找不到id而异常。

- 在onCreate中初始化值，更好的选择是自动设置默认值。

- 代码量很大，并且这些代码那一构建测试。

  



## 上手使用

-  **开启databind**

  在moudule下的build.gradle构建文件中的android块里开启

  ```xml
  android {
  ...
      dataBinding {
         enabled true
      }
  }
  ```

- **布局文件**

  - *增加根标签*

    需要把常规布局文件放在一个全局的Layout标签下，并在Layout标签下加Data标签，要显示的数据-布局变量就在这里声明

  ```xml
  <layout xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:app="http://schemas.android.com/apk/res-auto"
         xmlns:tools="http://schemas.android.com/tools">
     <data>
  		<variable name="name" type="String"/>
          <variable name="lastName" type="String"/>
          <variable name="index" type="Int"/>
     </data>
     <androidx.constraintlayout.widget.ConstraintLayout
             android:layout_width="match_parent"
             android:layout_height="match_parent">
  
         <TextView
  ...
  ```

  - *引用变量*

    使用布局表达式来引用声明的布局变量‘`@{`*expression*`}`’。例如

    ```xml
    android:text="@{String.valueOf(index + 1)}"
    android:tag="@{lastName}"
    android:visibility="@{age < 13 ? View.GONE : View.VISIBLE}"
    android:transitionName='@{"image_" + id}'
    ```

    > 应尽量避免使用过于负责的布局表达式

    表达式特性详情移步 [这里](https://zjcqoo.github.io/-----https://developer.android.com/topic/libraries/data-binding/expressions#expression_language)

- **Activity **

  在activity文件中填充view的方式有所更改，如

  ```kotlin
  val binding : PlainActivityBinding =
      DataBindingUtil.setContentView(this, R.layout.plain_activity)
  ```

​	 之后还要创建一些变量，在布局文件中使用到的我们在<data>块中声明的那些布局变量。这就是绑定对象                	 的作用。绑定类由库自动生成。



- **事件处理**

  替换变量为上级的对象变量。其中SimpleViewModel继承ViewModel，有name和lastname属性。

  ```xml
   <data>
      <variable
          name="viewmodel"
          type="com.example.android.databinding.basicsample.data.SimpleViewModel"/>
   </data>
  ```

  此时，布局表达式需这样写

  ```xml
   <TextView
      android:id="@+id/plain_name"
      android:text="@{viewmodel.name}"/>
  ```

​      *点击事件*：

​		首先在SimpleViewModel 编写实现 onLike()方法

​		此时，布局表达式需这样写：android:onClick="@{() -> viewmodel.onLike()}"



 - **数据观察**

   在某些情况下 我们要观察数据的变化。现在有两种方式可以实现观察数据的能力，[observable classes](https://zjcqoo.github.io/-----https://developer.android.com/topic/libraries/data-binding/observability?hl=es-419#observable_objects),[observable fields](https://zjcqoo.github.io/-----https://developer.android.com/topic/libraries/data-binding/observability?hl=es-419#observable_fields)  或是[LiveData](https://zjcqoo.github.io/-----https://developer.android.com/topic/libraries/data-binding/architecture?hl=es-419#livedata)

   - binding指定lifecycleOwner

     一般指定为当前的activitty/fragment

     ```kotlin
     binding.lifecycleOwner = this
     binding.viewModel = viewModel
     ```

   - 声明LiveData

     ```kotlin
     private val _name = MutableLiveData("Ada")
         private val _lastName = MutableLiveData("Lovelace")
         private val _likes =  MutableLiveData(0)
     
         val name: LiveData<String> = _name
         val lastName: LiveData<String> = _lastName
         val likes: LiveData<Int> = _likes
     ```

   - 数据改变

     ```kotlin
     fun onLike() {
             _likes.value = (_likes.value ?: 0) + 1
         }
     ```

   > 这里的例子只是某个值的观察，若要使一个对象可观察呢？ 官方文档 [observability](https://zjcqoo.github.io/-----https://developer.android.com/topic/libraries/data-binding/observability#java)



- **绑定适配器**

  上边的数据观察而自动展示最新的值是怎么做到的呢？ Data Binding让ui的几乎所有的调用都在Bind Adapter的静态方法中完成。Data Binding类库提供了大量的Bind Adapter，例如TextView Binding Adapter中的一个静态方法

  ```kotlin
      @BindingAdapter("android:text")
      public static void setText(TextView view, CharSequence text) {
          //view.text与text   对比
         	...
          view.setText(text);
      }
  ```

  现在我们来自定义一个Binding Adapter

  使用kotlin会很方便，建一个kotlin文件，编写方法（kotlin中会扫描工程文件把文件函数添加到顶层，创建静态方法作为类的扩展函数）

  在新建文件中编写如下方法

  ```kotlin
  //第一个参数View代表用于任何View
  //第二个参数number代表接收属性值
  @BindingAdapter("app:hideIfZero")
      fun hideIfZero(view: View, number: Int) {
          view.visibility = if (number == 0) View.GONE else View.VISIBLE
      }
  ```

  之后就可以给view添加该属性了

  ```xml
      <View
              app:hideIfZero="@{viewmodel.likes}"
              ...
  ```

  - *多参数Binding Adapter*

    ```kotlin
    /**
     *  Sets the value of the progress bar so that 5 likes will fill it up.
     *
     *  Showcases Binding Adapters with multiple attributes. Note that this adapter is called
     *  whenever any of the attribute changes.
     */
    @BindingAdapter(value = ["app:progressScaled", "android:max"], requireAll = true)
    fun setProgress(progressBar: ProgressBar, likes: Int, max: Int) {
        progressBar.progress = (likes * max / 5).coerceAtMost(max)
    }
    ```

- [samples](https://zjcqoo.github.io/-----https://github.com/googlesamples/android-databinding)
- [documentation](https://zjcqoo.github.io/-----https://developer.android.com/topic/libraries/data-binding/) 



