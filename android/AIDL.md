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


