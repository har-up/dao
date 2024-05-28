# AIDL
官方文档：https://developer.android.google.cn/develop/background-work/services/aidl?hl=zh-cn

aidl即Android Interface Define Language(Android 接口定义语言)。
它允许您定义客户端和服务端使用进程间通信(IPC)进行进程通信都遵循认可的编程接口。
在Android上，一个进程是无法访问另一个进程的内存的，为了实现进程交互，它们需要将对象分解成基元，以便操作系统可以识别这些基元，将其编组到该边界之外。
编写执行该编组的操作是一个繁杂的过程，因此Android使用AIDL帮我们处理。
需要注意的是，我们需要根据不同的场景选择是否选用AIDL。
* 当我们需要不同应用访问IPC服务，并且服务中需要处理多线程问题时适合选用AIDL。
* 如果不需要跨不同进程访，可以实现Binder来实现接口。通过Service的方式来和客户端通信
* 如果希望执行IPC,但不需要处理多线程问题，可以选用Messenger


## 编写AIDL文件
AIDL文件定义接口
src目录下新建aidl目录，在aidl目录下新建AIDL文件。
gradle assembleDebug 命令生成对应java文件

实现 AIDL 接口时，请注意以下几点规则：
* 传入的调用无法保证在主线程上执行，因此您需要从一开始就考虑多线程处理，并正确地将服务构建为线程安全服务。
* 默认情况下，IPC 调用是同步的。如果您知道服务完成请求所需的时间超过几毫秒，请不要从 activity 的主线程调用该服务，
  它可能会挂起应用，从而导致 Android 显示“应用无响    应”对话框。从客户端中的单独线程调用它。
* 只有 Parcel.writeException() 参考文档中列出的异常类型才会发回给调用方。

## 以服务的方式公开接口
编写一个服务类RemoteService集成Service。重写其onBind方法，返回服务的实现Binder对象。
```java
public class RemoteService extends Service {

    private static final String TAG = "Remote Service";

    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();


    final IRemoteService.Stub binder = new IRemoteService.Stub() {


        @Override
        public int getPid() throws RemoteException {
            return Process.myPid();
        }

        @Override
        public List<Book> getBookList() throws RemoteException {
            return mBookList;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            mBookList.add(book);
            Timber.tag("Service").d("add book success " + book.toString());

        }

        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }
    };

    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }
}
```

## 客户端请求服务端建立连接，获取服务端onBind（）返回的binder实例
客户端应用需要将相关的aidl文件和实体类文件全部从服务端拷贝过来，保持包名不变。
通过IRemoteService.Stub.asInterface(service) 将binder转换成声明的服务实例，从而访问服务提供的接口。
```kotlin
val serviceIntent = Intent().apply {
            action = "com.tiaomi.sign.REMOTE_SERVICE"
            setPackage("com.tiaomi.tools")
        }
        bindService(serviceIntent, object : ServiceConnection {
            override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
                mBookRemoteService = IRemoteService.Stub.asInterface(service)
                Toast
                    .makeText(
                        this@MainActivity,
                        "连接成功 ${mBookRemoteService?.pid}",
                        Toast.LENGTH_LONG
                    )
                    .show()
            }

            override fun onServiceDisconnected(name: ComponentName?) {
                Toast
                    .makeText(this@MainActivity, "连接断开", Toast.LENGTH_LONG)
                    .show()
            }
        }, Context.BIND_AUTO_CREATE)
```

## 服务的通知反馈
* 当服务端的数据变更需要及时同步到客户端时，需要创建aidl回调接口
```java
interface IBookEventListener {

    void onNewBookArrived(in Book newBook);
}
```

* 编写注册监听和注销监听方法
```java
 void registerListener(IBookEventListener listener);

 void unRegisterListener(IBookEventListener listener);

```
* 服务端监听注册和注销代码逻辑
用一个队列存放注册的listener,注销时则删除。
注意需要使用RmoteCallbackList,一般的队列无法识别出客户端注册的同一个listener。

* 注册IBinder连接死亡监听处理，释放资源或者重连等

* 权限验证
如果不做权限验证，我们的远程服务任何人都可以连接，这就存在着安全风险。
在AIDL中进行权限验证，通常使用的两种方法如下：
    * 在onBind中进行验证，如果验证不通过就直接返回null。
      1.服务端声明权限
      ```xml
      <permission android:name="com.tiaomi.tools.permission.ACCESS_BOOK_SERVICE"
        android:protectionLevel="normal"
        />
      <uses-permission android:name="com.tiaomi.tools.permission.ACCESS_BOOK_SERVICE"/>
      ```
      2.客户端声明权限
      ```xml
      <uses-permission android:name="com.tiaomi.tools.permission.ACCESS_BOOK_SERVICE"/>
      ```
      3.服务端代码中检测
      ```java
        int checkResult = PermissionChecker.checkCallingOrSelfPermission(
                this,"com.tiaomi.tools.permission.ACCESS_BOOK_SERVICE");
      ```
    * 在onTransact方法中验证，验证粒度更小些。
      也可以采用Permission的验证方式，具体实现和第一种方法类似。还可以采用Uid,Pid来做验证
    ```java
      getCallingPid();
      getCallingUid();
    ```
   
