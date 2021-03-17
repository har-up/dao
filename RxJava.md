## 目标
熟悉操作符
等待多个异步回调后继续处理的实现方案

# Rxjava
## 响应式编程
响应式编程是一种面向数据流的变化而传播的一种编程范式，传统的编程是按照顺序执行的，下一步的操作需要等待上一步的结果，这种编程方式清晰简洁，但还会有一个效率的问题。
试想多个步骤可以同时进行（异步执行）就可以节约很大的时间，响应式编程就是一种绝佳的编程方式。
响应式编程特点：
- 异步编程：提供合适的异步编程模型，充分挖掘多核cpu的能力，提升程序效率，降低延迟和阻塞。
- 数据流：基于数据流模型，提供一套接口，用于数据流的转化和传播订阅
## RxJava 简介 
### Rx的由来
Rx是Reactige Extensions的缩写，最初由LIQN的一个扩展，由微软的一个团队开发，目的是提供一个统一的接口帮助开发者更方便的处理异步数据流问题。
Rx库起初只支持NET,javascript,c++,应其实用性目前几乎所有的编程语言都有支持。

### Rx模式
- 创建： 方便的创建数据流
- 组合： 提供一系列接口对原始数据流进行转化
- 监听： 通过订阅来监听数据流

### RxJava Hello World
```java
  //核心元素   通过Observable 创建一个数据流（可观察对象） 一个观察者Consumer   订阅subscribe（进行关联）
  Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("Hello World");
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                HDLog.logD("TAG",s);
            }
        });
 Observable.just("hello world").subscribe(ToastUtils::showText);
```

## RxJava 基础
### Observable 
    可以发送0或n个数据源，以成功和错误事件结束
    Rxjava的使用通常需要三步
    - 创建Observable（可观察对象）
    - 创建Observer (观察者）
    - subscribe (订阅)，使Observable 和 Observer建立关联
      subscribe有多个重载的方法
    ```java
    subscribe(onNext);
    subscribe(onNext,onError);
    subscribe(onNext,onError,onComplete);
    subscribe(onNext,onError,onComplete,onSubscribe);
    ```
  在RxJava中，只有使用了subscribe()，被观察者才会开始发送数据。
#### Hot Observable 和 Cold Observable
     - Hot Observable无论有没有订阅者，事件都会发生，当有多个订阅者时，多个订阅者共同想用一个数据流信息
     - Cold Observbale只有设置了订阅者，才会开始执行发送数据流，且订阅者独享数据流
       有个很形象的比喻：Hot Observable像广播，听众听到的是同一个声音。Cold收音机，人们听到的是自己的收音机声音。
##### 二者的转化
      - publish() Cold Observable转变为Hot Observable,将原来Observable转变为ConnectableObservable
      - Subject/Processor Cold -> Hot。Subject和Processor的作用相同。Processor是rxjava 2.0新增的类，继承Flowable支持背压。
        > Subject及时Observable又是Observer,Rx官网称可以将Subject看做一个桥梁或代理
        > Subject 作为一个 Observable 时，可以不 地调用 onNext（）来发送事件，直至遇到 onComplete() 才会结束。
        > Subject的分类
          - SyncSubject
            Observer 会接 AsyncSubject onComplete（）之前的最后一个数据
          - BehaviorSubject
            Observer会先接收subject调用onComplete之前的最后一个数据（调用subject.onComplete之后才开始发射数据），然后接收之前的数据
          - ReplySubject
            ReplaySubject 会发射所有来自原始 Observable 的数据给观察者，无论它们是何时订阅的。  
          - PublishSubject
            Observer 只接收PublishSubject 被订阅之后发送的数据  
          
          
      - refCount ConnectableObservable的refCount可以将Hot Observable转化为 Cold Observable  
      - share share操作符封装了publish().refCount()操作
      
  
### Flowable
    Flowable是rxjava 2.x中新增的观察者，可以发送0或n个数据源，以成功和错误事件结束，支持背压，控制发射数据的速率
    一般使用场景：
    - 处理以某种方式产生超过10kb的元素
    - 文件读取与分析
    - 读取数据库记录
    - 网络io流
    - 响应式非阻塞接口
    

### Single
    只发射单个数据或错误事件，只有onSuccess和onError事件，onSuccess用于发射数据
### Completable
    不发送数据，只处理onComplete和onError事件
### Maybe
    可以发送0或1个数据，要么成功，要么失败
    
### do操作符
    do操作符可以给Observable生命周期的各个阶段增加回调，相当于生命周期钩子
    生命周期钩子有如下
    - doOnSubscribe 建立订阅时调用（最开始回调的钩子）
    - doOnLifeCycle 建立订阅后可以在该回调中取消
    - doOnNext Observable每发送一次数据就会触发该回调，它的Consumer接受发射的数据
    - doOnEach Observable每发送一次数据就会触发该回调，它的Consumer接受发射的数据，不仅onNext，还有onError,onComplete
    - doAfterNext 在onNext之后执行
    - doOnComplete Observable在正常终止时调用
    - doFinally Observable在正常或异常终止时调用
    - doAfterTerminate 注册一个Action,当Observable调用onComplete或onError时调用

## 创建操作符
  - just
    将一个或多个对象转换成发射这个或多个对象的Observable
  - from
    根据数据来源(Iterable,Future,数组)来转换成发射这些数据的Observable
  - create
    从头开始创建一个Observable
  - defer
    只有当订阅者订阅才创建Observable,为每个订阅者都创建一个Observable
  - range
    创建一个发射指定范围整数序列的Observable
  - interval
    创建一个按照一定的时间间隔发射整数序列的Observable
  - timer
    创建一个在给定的时间后发射单个数据的Observable
  - empty
    创建一个什么都不做，直接通知完成的Observable
  - error
    创建一个什么都不做，直接通知错误的Observable
  - never
    创建一个不发射任何数据的Observable
  


## Rxjava的线程操作
### Scheduler 调度器
- Rxjava 线程介绍
  Rxjava默认的是在当前线程工作，即在当前线程发射数据，也在当前线程进行监听处理。但为了更好的提升性能和速度，我们需要在其他线程做任务繁重的工作，只需在前台线程做监听就好。
  
- Scheduler
  Scheduler是一个抽象类，在Rxjava中作为线程控制器。Rxjava中也内置了多个实现：
  - single 单个线程重复利用
  - newThread 每次启动一个新的线程
  - computation 使用固定的线程池
  - io  适合io操作（读写文件，网络请求，读写数据库等），io内部实现是一个无数量限制的线程池，可以重用空闲的线程
  - trampoline  直接在当前线程运行，如果当前线程有其他任务在运行则会先暂停其他任务
  - Schedulers.from 自定义一个Executor来作为调度器

## Rxjava的变换和过滤操作符
  ### map 和 flatmap
      map对每一个发射的数据做处理，返回处理的数据即可，flatMap用于返回一个Observable
  ### groupBy
      用于对发射的数据根据条件来进行分组
  ### buffer和window
      缓存给定数量的数据，指定数量的一并发射
  ### first和last
      只发射第一个或最后一个数据；或第一个最后一个满足条件的数据
  ### takeFirst和takeLast
      发射前n和或后n个数据
  ### skip和skipLast
      跳过前n个或后n个数据
  ### elementAt和ingoreElment
      忽略某个指定的数据，以及只发射指定的数据
  ### filter和distinct
      对数据进行去重或根据判断来警醒过滤
  ### debounce  
      给定的时间只发射一个数据，可以用于防止数据量发射太快
  

## 条件操作符和布尔操作符
### boolean操作符
    boolean操作符可以判定observable发射的数据是否满足某一个条件，使得observable的观察者可以得到一个boolean类型回调。
    - all
      是否observable发射的所有数据都满足条件
    - contains
      observable发射的数据是否包含某个值
    - amb
      针对多个observable,只发射首先发送给amb的observable的所有数据。
      例如observable a 发射 1 2 3；observable b 发射 4 5 6。使用amb操作符后可以是先发射a的数据也可以发射b的数据，但只发射a,b中的一个。
    - sequenceEqual
      判定两个Observable是否发射相同的数据 
### 条件操作符
    - defaultEmpty
      如果Observable没有发射任何数据则发射一个给定的默认的值
    - skipUtil
      跳过发射Observable a的数据直到Observable b发射满足某个条件时
    - skipWhile
      和skipUtil相反，跳过发射Observable a的数据 直到 Observable b不满足某个条件
    - takeUtil
      Observable a发射数据，但只要Observable b发送了一个数据或通知则a不再发射数据
    - takeWhile
      Observable a发射元素数据，直到发射的数据不满足某个条件

## 合并操作符和连接操作符
### 合并操作符
    - merge
      合并两个Observable为一个Observable发射它们的所有数据，这些数据按顺序发射
    - zip
      将多个Observable的发射值结合在一起
### 连接操作符
    - combineLastest
      有点类似于merge,zip是只有当原始的Observable中的每一个都发射了一条数据时才发射数据，任何一个Observable发了一条数据时， combineLatest使用
      个函数结合它最近发射 数据，然后发射这个函数返回值。
    - join
      将多个Observable的发射值以交集的情况发射组合数据
    - startWith
      将Observable的开头插入一个指定的数据
    - connect
    - push
    - refCount
    - reply
    对于连接操作符，有一个很重要的概念 connectablObservable--可连接的 Observable 在被订阅时并不发射数据，只有在它的 connect（）被调用时才开始发射数据。

## 背压
   当被观察者发送数据量过大时，观察者来不及响应处理数据，这就是背压的情况
   Rxjava1中当发射的数据量大于16时就会报背压的错误
   Rxjava2中使用Flowable来支持背压，默认支持128的数据量。
   在Rxjava2的Flowable中可以通过配置背压策略来使用现实场景
   - MiSSING
   - Error
     直接报错
   - Buffer
     可以无限发射不会报背压的错误，但会导致OOM
   - Drop
     如果异步缓存池满了，则丢弃后面发射的数据
   - Latest
     丢弃已经放入缓存池中的数据
