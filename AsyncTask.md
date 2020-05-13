## AsyncTask

AsyncTask是一个Android中用于多线程任务类。可以通过实现类（必须实现doInBackground方法，该方法在非主线程中运行）对象的execute(param)执行异步任务。

## 

### 原理 

​	AsyncTask有两个方法可以开启异步任务,分别是execute，excuteOnExecutor。execute内部也是调用的executeOnExecutor，因为AsyncTask内部提供了一个默认的Exector,这个Exector是内部定义的一个静态内部类SerialExecutor。

- executeOnExecutor

  - 先判断当前这个AsyncTask对象是否在执行状态或执行完成，然后调用其抽象方法onPreExecute（在真正执行线程任务前执行的回调);

  - 将AsyncTask对象的FutureTask对象交与 SerialExecutor执行线程任务。FutureTask对象的构造需要一个Callable对象。在AsyncTask中是mWorker

    ```java
    mWorker = new WorkerRunnable<Params, Result>() {
                public Result call() throws Exception {
                    mTaskInvoked.set(true);
                    Result result = null;
                    try {
                        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                        //noinspection unchecked
                        result = doInBackground(mParams);
                        Binder.flushPendingCommands();
                    } catch (Throwable tr) {
                        mCancelled.set(true);
                        throw tr;
                    } finally {
                        postResult(result);
                    }
                    return result;
                }
            };
    ```

    

    由上面的代码可知，AsyncTask的doInBackground内需要执行的操作就放在到Callable对象的的call（)方法中执行。

  - 执行的进度和结果又是怎么返回到主线程的呢？

    FutureTask的机制中，在执行Callable中的call（）完毕后，可以通过get()获取其返回值，如果还没完成则会阻塞等待其完成。当成功获取到返回值后就通过getMainLooper()获取的主线成Looper构造的handler去调用onPostExecute（）。至于进度则是需要自己在doInBackground中显示调用publishProgress()，然后在onProgressUpdate（）中接收回调。

### SerialExecutor

属性:

- 定义了一个存放Runnable对象的双向队列Deque对象mTask
- Runnable对象mActive,用于执向在执行的runnable;			

实现的Executor接口，即实现其execute(Runnable runnable)方法。他的内部逻辑是重新包装runnable（新建一个Runnable对象，该Runnable对象直接执行接收到的runnable的run方法）。把新的Runnable对象添加到双向队列mTask(尾部插入),而后判断mActive是否为NULL，如果为NULL则从双向队列的头部删除并取出一个runnable，取出的runnable交与Async中默认的ThreadPoolExecutor对象去真正开启线程执行操作。