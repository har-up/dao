# EventBus
EventBus是一个简单易用的用于组件间通信的开源库
## 优点
- 解耦事件触发和事件接收
- 在ui展示及后台线程上性能表现良好
- 避免复杂和容易出错的依赖问题以及生命周期方面的问题
- 库的体积小 60k左右
- 高级特性，线程传递/订阅优先级

## 特性
- 基于注释的API,使用方便
- 主线程事件传递
- 后台线程事件传递
- 事件订阅者继承  事件类B继承事件类A，当post事件类b是，订阅事件A的订阅者也会接收到

## 整合流程
  - 添加依赖
  ```gradle
  implementation 'org.greenrobot:eventbus:3.2.0'
  ```
  
  - 定义事件类
  ```java
  public class MessageEvent {
    public final String message;
    public MessageEvent(String message) {
        this.message = message;
    }
  }
  ```
  
  - 注册订阅者
  ```java
  // This method will be called when a MessageEvent is posted (in the UI thread for Toast)
  @Subscribe(threadMode = ThreadMode.MAIN)
  public void onMessageEvent(MessageEvent event) {//这里的参数对应事件类，那个事件类对象被post,
  那个事件类或是其父类作为参数的订阅者就会接收到事件回调
    Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
  }
  ```
  
  - 在声明周期组件中（activity/fragment）注册/解除 注册
  ```java
  @Override
  public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
  }
 
  @Override
  public void onStop() {
    EventBus.getDefault().unregister(this);
    super.onStop();
  }
  ```
  
  - post事件对象
  ```java
  EventBus.getDefault().post(new MessageEvent("Hello everyone!"));
  ```
  
  ## 线程间传递
  EventBus中事件的发送和事件的接收可以不在同一线程，可以通过TheadMode的几个不同属性来配置不同模式。
  - ThreadMode.POSTING（默认模式）
    事件的接收需要和事件的发送在同一线程。这种情况下开销最小，因为没有线程的切换开销，
  - ThreadMode.Main
    事件的接收处理在主线程，需要注意的是事件的处理不能有耗时操作以避免阻塞。
  - ThreadMode.MAIN_ORDERED
    事件的接收处理在主线程，多个事件post时按照相应的顺序串行执行
  - ThreadMode.BACKGROUND
    事件的接收处理在后台线程，事件post线程是同步等待的，也不能有太耗时的操作
  - ThreadMode.ASYNC
    事件的接收处理在一个单独的线程执行，并且事件post线程与处理线程异步执行，可以在接收处理方法中执行一些耗时操作
    
  ## Configuration
  可以通过其Builder类对EventBus的一些执行策略进行配置
  ```java
  EventBus eventBus = EventBus.builder()
    .logNoSubscriberMessages(false)
    .sendNoSubscriberEvent(false)
    .build();
  ```
  配置好后可以将该实例作为默认实例
  ```java
  eventBus.installDefaultEventBus();
  ```
  
  ## 粘性事件
  可以使用粘性事件来替代一些数据的缓存,因为EventBus有对事件做缓存的功能。比如打开一个新的activity,我需要加载保存的位置信息。
  在EventBus中可以这样用
  - post粘性事件
  ```java
  	EventBus.getDefault().postSticky(new MessageEvent("Hello everyone!"));
  ```
  - 新打开的activity中接收粘性事件
  ```java
  // UI updates must run on MainThread
  @Subscribe(sticky = true, threadMode = ThreadMode.MAIN)
  public void onEvent(MessageEvent event) {   
    textField.setText(event.message);
  }
  ```
  - 粘性事件的管理
  ```java
  MessageEvent stickyEvent = EventBus.getDefault().getStickyEvent(MessageEvent.class);
  // Better check that an event was actually posted before
  if(stickyEvent != null) {
    // "Consume" the sticky event
    EventBus.getDefault().removeStickyEvent(stickyEvent);
    // Now do something with it
  }
  ```
  
  ## 订阅优先级和订阅取消
  - 在有多个订阅者的情况下，可以通过设置订阅优先级来调整调用顺序。在相同的模式ThreadMode下，
    拥有高优先级的比订阅者先执行。
    ```java
    @Subscribe(priority = 1);//默认为0
    public void onEvent(MessageEvent event) {
    }
    ```
  - EventBus提供了事件取消的接口，当取消某个事件的传递后，事件订阅者将不再接收到该事件。一般会在高优先级的
  订阅中去取消低优先级的事件，需要注意的是，只能在线程处理方法中取消该线程中的post事件。
    ```java
    // Called in the same thread (default)
    @Subscribe
    public void onEvent(MessageEvent event){
      // Process the event
      ...
      // Prevent delivery to other subscribers
      EventBus.getDefault().cancelEventDelivery(event) ;
    }
    ```
  ## 使用订阅者索引
  在EventBus 3.0可以使用订阅者索引从而避免昂贵开销的反射，其原理是java的注释处理器，所以需要引入注释处理。
  - 引入java/kotlin注释处理
  ```java
  android {
      defaultConfig {
          javaCompileOptions {
              annotationProcessorOptions {
                  arguments = [ eventBusIndex : 'com.example.myapp.MyEventBusIndex' ]
             }
          }
      }
  }
 
  dependencies {
      def eventbus_version = '3.2.0'
      implementation "org.greenrobot:eventbus:$eventbus_version"
      annotationProcessor "org.greenrobot:eventbus-annotation-processor:$eventbus_version"
  } 
  ```
  ```kotlin
  apply plugin: 'kotlin-kapt' // ensure kapt plugin is applied
 
  dependencies {
      def eventbus_version = '3.2.0'
      implementation "org.greenrobot:eventbus:$eventbus_version"
      kapt "org.greenrobot:eventbus-annotation-processor:$eventbus_version" 
  }
 
  kapt {
      arguments {
          arg('eventBusIndex', 'com.example.myapp.MyEventBusIndex')
      }
  }
  ```
  
  - build自动生成相应的类
  ```java
  EventBus.builder().addIndex(new MyEventBusIndex()).installDefaultEventBus();
  // Now the default instance uses the given index. Use it like this:
  EventBus eventBus = EventBus.getDefault();
  ```
  
  
  
