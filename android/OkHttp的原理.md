# OkHttp的原理
## execute同步请求
基本使用方法：
```kotlin
   var okHttpClient = OkHttpClient()
   var newCall = okHttpClient.newCall(Request.Builder().build())
   newCall.execute()
```
可以知道真正胡执行过程是newCall的execute（）方法。newCall是一个RealCall类型，所以进入到RealCall的execute();
```java
  @Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    try {
      client.dispatcher().executed(this);
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } catch (IOException e) {
      eventListener.callFailed(this, e);
      throw e;
    } finally {
      client.dispatcher().finished(this);
    }
  }
```
里边的逻辑就是先判断该RealCall对象是否已经执行，未执行的话交给okhttpclient的dispatcher管理，然后执行getResponseWithInterceptorChain()。
首先了解一下Dispatcher类，这是一个任务调度类，他有个三个队列readyAsyncCalls、runningAsyncCalls、runningSyncCalls通过名字可以看出他们的
作用。
  - runningAsyncCalls 存放异步执行中的请求
  - readyAsyncCalls 存放待执行的请求，因为Dispatcher有请求数的限制，默认是64，如果正在执行的请求为等于这个数，那么就先将其放入这个队列
  - runningSyncCalls 执行的同步请求，与Dispatcher的数量限制无关。
同步请求的流程就是这么简单，因为没有涉及到线程池。getResponseWithInterceptorChain做了些什么工作后面在分析。

## execute异步请求
基本使用方法：
```kotlin
   var okHttpClient = OkHttpClient()
   var newCall = okHttpClient.newCall(Request.Builder().build())
   newCall.enqueue(callback)
```
异步请求是调用RealCall对象的enqueue()并入参一个回调方法用于接收执行完成后的数据。
```java
  @Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }
```
enqueue方法很简单，判断realcall是否已经执行，没有就将其包装成AsyncCall对象加入到异步队列
```java
  synchronized void enqueue(AsyncCall call) {
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      runningAsyncCalls.add(call);
      executorService().execute(call);
    } else {
      readyAsyncCalls.add(call);
    }
  }
```
当成功加入到异步请求队列后，接着调用 executorService().execute(call);executorService()是Dispatcher类中的一个方法，返回一个线程池
该线程池只有非核心线程，如下
```java
public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }
```
线程池执行的是Runnable，而包装成胡AsyncCall就是继承的Runnable（AsyncCall继承NamedRunnable，NamedRunnable是实现Runnable接口胡一个抽象类）
```java
public abstract class NamedRunnable implements Runnable {
  protected final String name;

  public NamedRunnable(String format, Object... args) {
    this.name = Util.format(format, args);
  }

  @Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

  protected abstract void execute();
}
```
所以后边线程池中线程执行的是AsyncCall的execute（）方法。如下：
```java
    @Override protected void execute() {
      boolean signalledCallback = false;
      try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this, response);
        }
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          eventListener.callFailed(RealCall.this, e);
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
    }
```
可以知道其中也是通过getResponseWithInterceptorChain()进行真正的请求。


## getResponseWithInterceptorChain()
看其中的源码：
```java
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
  }
}
```
新建一个列表，加入自己定义的interceptors后再加入Okhttp所需要的5个拦截器分别是RetryAndFollowUpInterceptor、
BridgeInterceptor、CacheInterceptor、ConnectInterceptor、CallServerInterceptor。这些拦截器通过拦截器链连接起来，会依次调用
各个拦截器的intercept方法。责任链模式——大致的代码如下
```java
interface Chain{
    void procced();
}

class RealChain implements Chain{
    private List<Interceptor> interceptors;
    int index;
    public RealChain(List<Interceptor> interceptors,int index){
        this.interceptors = interceptors;
        this.index = index;
    }

    @Override
    public void procced() {
        if(index >= interceptors.size()){
            return;
        }
        RealChain next = new RealChain(interceptors, index + 1);
        Interceptor interceptor = interceptors.get(index);
        interceptor.intercept(next);
    }
}

interface Interceptor{
    void intercept(Chain chain);
}

class InterceptiorA implements Interceptor{
    @Override
    public void intercept(Chain chain) {
        ...
        chain.procced();
        ...
    }
}

class InterceptiorB implements Interceptor{
    @Override
    public void intercept(Chain chain) {
        ...
        chain.procced();
        ...
    }
}
```
以链为参数渗透每一个拦截器，每次调用链chain的procced就会重新创建一个新的chain，然后把新胡chain传递给下一个拦截器。

### 系统的几个拦截器
- RetryAndFollowUpInterceptor 失败重试拦截器，在这个拦截器里边有个循环，如果请求出错就不断重试，初始化streamAllocation。
- BridgeInterceptor 桥接拦截器，这里主要定义一些头信息
  - Content-Type 如果请求体中的Content-Type不为null,则添加对应值到header
  - Content-Length 请求体大小，如果没有对应值则设置Transfer-Encoding"-"chunked"（代表分块传输，当传输的块为空时代表传输完毕）
  - Host 请求的host
  - Connection 是否长连接
  - User-Agent 请求方请求工具信息
- CacheInterceptor 做缓存处理
- ConnectInterceptor 通过streamAllocation初始化httpCodec、connection。主要通过这几个对象实现流的读写。
- CallServerInterceptor 执行真正的连接传输获取数据。通过httpCodec的两个对象进行读写，HttpCodec有两个实现类分别支持http1.1、http2
  ```java
    //okio包下的类
    final BufferedSource source; //读
    final BufferedSink sink; //写
  ```

