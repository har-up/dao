## Dialog
有一个统一的Dialog样式对App是极其重要的，这种统一规范越早做越好，开发者应该尽早调用统一封装好的java api。
- Dialog和Window
Dialog初始化的过程：
  - context的初始化，通过传入的Activity context获取dialog的主题资源
  - 通过Activity context获取WindowManager,建立PhoneWindow，设置dialog的回调监听

- show 和 dismiss方法
显示一个Dialog的过程其实就是将View挂载到Window的过程，在方法执行完毕后Dialog会发送一个已经显示的信号用来标记当前的dialog的状态并触发监听事件。
在Android中只要涉及到View的展示，必然会有View数据保存相关的问题，在Dialog中也提供了类似于Activity的数据保存方法。Dialog中的onSaveIstanceState()
和onRestoreInstanceState配合可以实现view状态的保存与恢复，但当dialog中有自定义view时，需要我们做相应的处理才能保存状态。

## AlertDialog
AlertDialog 是 Dialog的子类，通过其构造方法和他所有的重要逻辑都是交个AlertController进行代理的。
- AlertController
