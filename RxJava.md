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
### RxJava
#### Rx的由来
Rx是Reactige Extensions的缩写，最初由LIQN的一个扩展，由微软的一个团队开发，目的是提供一个统一的接口帮助开发者更方便的处理异步数据流问题。
Rx库起初只支持NET,javascript,c++,应其实用性目前几乎所有的编程语言都有支持。

#### Rx模式
- 创建： 方便的创建数据流
- 组合： 提供一系列接口对原始数据流进行转化
- 监听： 通过订阅来监听数据流

#### RxJava Hello World
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
