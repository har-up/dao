# Glide
Glide是一个快速高效的Android图片加载库，注重于平滑的滚动。
Glide提供了易用的API，高性能、可扩展的图片解码管道（decode pipeline），以及自动的资源池技术。
Glide注重Android加载图片的两个关键性能点
- 图片解码熟读
- 图片解码时产生垃圾数据，占用内存
Gilde采用多个步骤使Android加载图片尽可能快及平滑
- 智能和自动的减采样，最小缓存开销，解码时间
- 尽可能的资源重用，比如字节数组/bitmap，从而最小话垃圾回收和碎片资源整理
- 吻合android声明周期组件，优先处理处于活跃状态的activity和fragment。

## Gilde的集成步骤
- SDK版本要求
  - Minimum SDK Version： 14或更高
  - Compile SDK Version： 27或更高
  - Support Library Version： 27
  支持库若想要其他版本也可以配置，如下
  ```gradle
  dependencies {
    implementation ("com.github.bumptech.glide:glide:4.11.0") {
      exclude group: "com.android.support"
    }
    implementation "com.android.support:support-fragment:26.1.0" //使用26的版本
  }
  ```
- Gradle添加依赖
 ```gradel
 repositories {
  mavenCentral()
  maven { url 'https://maven.google.com' }
 }

 dependencies {
    compile 'com.github.bumptech.glide:glide:4.11.0'
    // Skip this if you don't want to use integration libraries or configure Glide.
    annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
 }
 ```
 
 ## Android权限配置
 如果应用程序只用到应用内存，那么不需要配置权限。但多数应用需要把数据存储在sd或其他应用之外的内存，
 就要进行一些权限配置。
 - Internet 网络访问权限
 ```xml
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="your.package.name"
    <uses-permission android:name="android.permission.INTERNET"/>
    <!--
    Allows Glide to monitor connectivity status and restart failed requests if users go from a
    a disconnected to a connected network state.
    -->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <application>
      ...
    </application>
  </manifest>
 ```
 - 网络连接监听
 Glide会自动监听网络变化情况。比如在请求网络图片的时候断网了，在网络恢复后，
 Glide监听到这些网络变化会继续之前的请求。关键日志："ConnectivityMonitor "
 
 - 本地存储
 为了可以访问本地存储，如系统DCIM/PICTURES文件夹下的数据，需要添加权限
 ```xml
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
 ```
 为了让Glide中的ExternalPreferredCacheDiskCacheFactory类存储缓存到本地，需要添加写入权限
 ```xml
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
 ```
 
 ## 基本的使用
 - 通过url加载网络图片
 ```java
 Glide.with(fragment)
    .load(myUrl)
    .into(imageView);
 ```
 - 取消加载
 ```java
 Glide.with(fragment).clear(imageView);
 ``
 - 加载请求定制
 Gradle提供了丰富的接口让你可以定制个性化请求，比如转换，缓存等配置
 一般个性化请求可以直接用调用链
 ```
 Glide.with(fragment)
  .load(myUrl)
  .placeholder(placeholder)
  .fitCenter()
  .into(imageView);
 ```
 如果这样的定制化请求在多个地方使用到，可以是用一个定制化类处理
 ```java
  RequestOptions sharedOptions = new RequestOptions()
     .placeholder(placeholder)
     .fitCenter();

  Glide.with(fragment)
    .load(myUrl)
    .apply(sharedOptions)
    .into(imageView1);
 ```
 
 ## 配合ListView和RecyclerView的使用
 在ListView和RecyclerView中加载使用和基本的加载一样
 ```java
 @Override
 public void onBindViewHolder(ViewHolder holder, int position) {
    String url = urls.get(position);
    Glide.with(fragment)
        .load(url)
        .into(holder.imageView);
 }
 ```
 有一些需求是，在之前已经加载过的位置重新加载渲染一图片，需要用到clear（）接口
 ```java
 @Override
 public void onBindViewHolder(ViewHolder holder, int position) {
    if (isImagePosition(position)) {
        String url = urls.get(position);
        Glide.with(fragment)
            .load(url)
            .into(holder.imageView);
    } else {
        Glide.with(fragment).clear(holder.imageView);
        holder.imageView.setImageDrawable(specialDrawable);
    }
 }
 ```
 
 ## 加载到非View目标
 该功能还不够完善，若要使用到该功能需要仔细阅读[文档](http://bumptech.github.io/glide/doc/targets.html)
 ```java
 Glide.with(context
  .load(url)
  .into(new CustomTarget<Drawable>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<Drawable> transition) {
      // Do something with the Drawable here.
    }

    @Override
    public void onLoadCleared(@Nullable Drawable placeholder) {
      // Remove the Drawable provided in onResourceReady from any Views and ensure 
      // no references to it remain.
    }
  });
 ```
 
 ## 后台线程加载
 关键接口：commit()
 ```java
 FutureTarget<Bitmap> futureTarget =
  Glide.with(context)
    .asBitmap()
    .load(url)
    .submit(width, height);

 Bitmap bitmap = futureTarget.get();

 // Do something with the Bitmap and then when you're done with it:
 Glide.with(context).clear(futureTarget);
 ```

## 占位图的使用
占位图可以分三种情况
- placeholder
  图片请求中的占位图
- error
  图片请求失败的占位图
- fallback
  图片请求url为null时的占位图
  
## 个性化定制接口
Glide的大多数个性定制化接口都通过RequestBuilder对象配置（即Glide.with()返回的对象);
获取RequestBuilder对象
```java
//1
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).asDrawable();
//2
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).load(url);
```

项目中用到的多为下面几个配置
- Placeholders 占位图
- Transformations 图片转换
- Caching Strateies
- 组件相关配置

