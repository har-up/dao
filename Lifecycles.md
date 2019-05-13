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

  使用ViewModel使数据状态更持久