# Lifecycles

## 介绍

使用生命周期感知的组件来处理生命周期的改变。当一个组件（activit/fragment）的状态发生改变时，生命周期感知组件会随这这个变化而产生不同的响应动作，这样的组件可以使我们的代码更有组织的，轻量级，更易掌控。

用于构建生命周期感知组件的几个重要对象

- **ViewModel**

  

- **LifecycleOwner/LifecycleRegistryOwner**

- **LiveData** 

## [依赖库](https://zjcqoo.github.io/-----https://developer.android.google.cn/jetpack/androidx/releases/lifecycle#declaring_dependencies)

现在maven 仓库中加google()

```xml
repositories {
    google()
    jcenter()
}
```

```xml
def lifecycle_version = "2.0.0"

    // ViewModel and LiveData
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
    // alternatively - just ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version" // For Kotlin use lifecycle-viewmodel-ktx
    // alternatively - just LiveData
    implementation "androidx.lifecycle:lifecycle-livedata:$lifecycle_version"
    // alternatively - Lifecycles only (no ViewModel or LiveData). Some UI
    //     AndroidX libraries use this lightweight import for Lifecycle
    implementation "androidx.lifecycle:lifecycle-runtime:$lifecycle_version"

    annotationProcessor "androidx.lifecycle:lifecycle-compiler:$lifecycle_version" // For Kotlin use kapt instead of annotationProcessor
    // alternately - if using Java8, use the following instead of lifecycle-compiler
    implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

    // optional - ReactiveStreams support for LiveData
    implementation "androidx.lifecycle:lifecycle-reactivestreams:$lifecycle_version" // For Kotlin use lifecycle-reactivestreams-ktx

    // optional - Test helpers for LiveData
    testImplementation "androidx.arch.core:core-testing:$lifecycle_version"
```

## 上手使用
 - **使用ViewModel**
  使用ViewModel使数据状态更持久,当Activity因转换横竖屏虽然会重新销毁创建，但中间ViewModel的不会销毁.
    ![](https://codelabs.developers.google.com/codelabs/android-lifecycles/img/1d42e8efcb42ff58.png)
  
 - **使用LiveData**
    
    - 创建LiveData对象
    
      ```java
      private MutableLiveData<Long> mElapsedTime = new MutableLiveData<>();
      ```
    
    -  值改变时发布改变信号
    
       当创建的LiveData对象的值发生改变时，发布改变的信号
    
       ```java
       mElapsedTime.postValue(newValue);
       ```
    
    -  订阅
    
       在Activity/Fragment中订阅，当订阅的值发生改变时进行相关处理（更新UI等）
    
       ```java
       private void subscribe() {
           	//创建观察者
               final Observer<Long> elapsedTimeObserver = new Observer<Long>() {
                   @Override
                   public void onChanged(@Nullable final Long aLong) {
                       String newText = ChronoActivity3.this.getResources().getString(
                               R.string.seconds, aLong);
                       ((TextView) findViewById(R.id.timer_textview)).setText(newText);
                       Log.d("ChronoActivity3", "Updating timer");
                   }
               };
           //LiveData对象被订阅  this的生命周期内为订阅时间，elapsedTimeObserver为观察着
       	mLiveDataTimerViewModel.getElapsedTime().observe(this,elapsedTimeObserver);
        }
       ```
    
- **订阅生命周期事件**

    LifecycleOwner对象可以获取当前所处的生命周期状态

    ```java
    lifecycleOwner.getLifecycle().getCurrentState()  //得到的值为 Lifecycle.State.RESUMED, or Lifecycle.State.DESTROYED 等等
    ```

    

    生命周期感知组件实现LifecycleObserver可以作为观察者订阅LifecycleOwener对象。

    比如实现LifecycleObserver接口的Activity/Fragment中可以这样用

    ```java
    lifecycleOwner.getLifecycle().addObserver(this); //this instanceOf LifecycleObserver
    ```

    然后使用注释的方式指定方法的触发条件

    ```java
    @OnLifecycleEvent(Lifecycle.EVENT.ON_RESUME)
    void addLocationListener() { ... }
    ```

- **Fragment间共享ViewModel**

  在一个Activity，多个Fragment的情况下，可以实现共享一个ViewModel的需求。在fragment中获取viewmodel时是同一个对象。所以可以达到共享ViewModel的目的。



- **ViewModel持久化，程序切换共享**

  编写ViewModel的有参构造器，形参为SavedStateHandle

  ```
  public SavedStateViewModel(SavedStateHandle savedStateHandle) {
     mState = savedStateHandle;
  }
  ```
