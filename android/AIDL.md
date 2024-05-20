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