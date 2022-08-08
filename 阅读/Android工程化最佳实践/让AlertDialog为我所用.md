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
AlertController的构造方法中初始化了有很多布局对象,这些布局最终会被填充到Dialog提供的白板中
比如: mAlertDialogLayout(title,message,button,容器)、mButtonPanelSideLayout(三个按钮在右侧的容器）
mListLayout(AlertController.RecyclerListView)、mMultiChoiceItemLayout(CheckeTextView)
详细的布局可以参考源码中theme的定义。
> 我们可以使用自定义theme设定对应的布局文件来达到自定义dialog的目的


- AlertDialog.Builder
上边可以知道布局是通过AlertController设置的，那么个这些布局中的View填充数据就是AlertDialog.Builder的工作了
因为Dilog中的每个数据都可以独立出自，对于有大量可选数据的对象，在java中一般通过buider模式构建。AlertDialog.Builder构架的数据会存到
AlertParams中，AlertParam.apply（）执行了最后的装配工作，将数据分别设置到Dialog的布局的View中。
> AlertDialog自身的theme是通过alertDialogTheme进行设置的，我们可以在style中通过设置如下的属性定义它的样式
  ```java
  <item name="alertDialogTheme">@style/Theme.Dialog.Alert</item>
  ```

## DialogFragment
AlertDialog可以通过简单的配置来实现弹框，但需求中很多的弹框比较复杂，比如对传入的参数做校验，对输入做判空校验，做一些通用的背景设置，进行网络请求并对数据做
处理等等，如果都用AlertDialog来做，上述的逻辑实现都需要放在Activity中完成，会导致Activity中的代码变得混乱，臃肿，也让Dialog失去内聚性。
我们可以通过Dialog帮助类解决重复代码过多和内聚性差的问题，但无法解决Dialog数据保存、生命周期管理等问题。
所以Google在Android 3.0中引入了一个新类dialogFragment。我们可以通过DilogFragment作为Controller管理AlertDialog。
- Fragment和Dialog
目前，官方推荐使用dialogFragment管理对话框，它可以确保正确处理生命周期时间。DialogFragment就会一个Fragment,当用户旋转屏幕时仍然是进行Fragment的执行过程
> 旋转屏幕需要做Dialog的数据保存和恢复的工作。
虽然DialogFragment是Dialog，但用法和Fragment不太一样，google推荐做法是不要在onCreateView中直接inflate一个布局，而是在onCreateDialog中建立Dialog对象

- show 和 dimiss
show和dismiss是dialog中国的两个重要方法，在DialogFragment中，这两个方法会被Fragment间接调用。
show：DialogFragment的show方法会建立一个Fragment对象，然后执行无容器的add操作。Fragment启动后会调用内部的onCreateDialog建立真正的Dialog对象，最后在onStart()中
触发dilaog的show（）方法。
> 需要注意必须在执行onStart之后才能指定dialog.findView()操作，否则会报空异常

dismiss: DialogFragment提供了两个关闭的方法dismiss和dissmissAllowingStateLoss(),前者对应fragmentTransaction.commit，后者对应fragmentTransaction.commitAllowingStateLoss。使用后者的好处是可以忽略在异步关闭Dialog时的状态问题，不用考虑当前Activity的状态，减少线上崩溃的次数。


## 实际问题
- 无法弹出输入框
当自定义dialog中有edittext时，希望弹出Dialog后能自动弹出输入法，但原生Dialog并不支持这种操作。
可以在DialogFragment中的onStart（）后调用如下代码，强制弹出输入法
```java
public void showInputMethod(final EditText editText){
    editText.post(new Runnable{
        @Override
        public void run(){
            editText.setFocusable(true);
            editText.setFocusableInTouchMode(true);
            editText.requestFocus();
            InputMethodManager imm = (InputMethodManager)getActivity().getSystemService(Context.INPUT_METHOD_SERVICE);
            if(imm != null){
                imm.showSoftInput(editText,InputMethodManager.SHOW_IMPLICIT);
            }

        }
    });
}
```

- 如何支持层叠弹窗
利用Fragment的回退栈完成

- 容易引起内存泄漏
  - dialog做了消失动画，动画未完成，Activity被finish
    在被finish后，取消动画，置空
  - 主线程looperhuoqu message获取到dismiss dialog的消息
    在handlerThread空闲时给队列发送一个null的message
    ```java
    static void flushStackLocalLeaks(Looper looper){
        final Handler handler = new Handler(looper);
        handler.post(new Runnable{
            @Overide
            public void run(){
               Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler(){
                @Overide
                public boolean queueIdle(){
                    handler.sendMessageDelayed(handler.obtainMessage(),1000);
                    return true;
                }
            }  
            }
        });
    }

- 修改尺寸，背景和动画
在onStart中修改window属性。可以设置标题，宽高，位置，动画（动画也可以在style中配置）

- 点击后会自动关闭
替换listener

- 关闭或开启时崩溃
> can not perform this action after onSaveInstanceState
1.try-catch
2.状态判断
3.commitAllowingStateLoss

## 封装Dialog
- 三方库 easyDialog

- 设置全局样式
修改Dialog的样式

- drawable inset的使用

- 可全局弹的Dialog
  Application持有当前Activity的引用。