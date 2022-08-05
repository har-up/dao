## 源码分析
- 静态变量以类型为前缀，如果多个成员开发同一个类，可以将常量或成员变量定义在逻辑需要的地方，不一定放在类的前边
- 使用注解代替枚举 @IntDes @StringDef
- Intent的深拷贝
  Inteng实现了cloneable接口，其clone方法为一个深拷贝
  其中有一个参数mSourceBounds是一个矩形对象，利用这个参数可以很方便的在activity间传递位置信息，甚至可以共享元素动画
  > Android系统默认会讲用户点击的“桌面图标”的位置发送给mainActivity,以此实现某些系统的过渡动画，所以在mainActivity中是可以获取当前App的icon坐标
    ```java
    Rect sourceBounds = getIntent().getSourceBounds();
    sourceBounds.toShortString()
    ```
- makeMainActivity
  ```java
  Intent intent = Intent.makeMainActivity(new ComponentName(getApplication(), MainActivity.class));
  ```
- makeRestartActivityTask
  主要用来重新启动主页面，多用在通知推送的场景下。
  在推送场景下，点击进入目标页面后后退希望回到主页面，意味着点击推送消息后等于启动多个activity；PendingInteng正好提供了一个getActivityes（），可以设置
  一个intent数组，用来指定一系列的activity
  ```java
  Intent[] intents = new Intent[2];
  intents[0] = Intent.makeRestartActivityTask(new ComponentName(getApplication(), MainActivity.class));
  intents[1] = Intent.makeRestartActivityTask(new ComponentName(getApplication(), CardDetailActivity.class));
  if (isMainActivityActive){
    getApplicationContext().startActivity(intents[1]);
  }else{
    startActivities(intents);
  }
  ```
  > 该方法会清空任务栈，app在前台的时候谨慎使用

- Intent的Chooser
  Android上的隐式意图可以找到多个可以处理该意图的组件
  比如:
  ```java
  Intent intent = new Intent();
  intent.addCategory(Intent.CATEGORY_APP_BROWSER);
  intent.setData(uri);
  Intent.createChooser(intent,"请选择浏览app");
  ```
  > 系统采用的是intent包裹intent做法，外层intent指向选择界面
    queryIntentActivities可以让我们知道哪些activity可以处理Intent

- URI代替Intent
  ```java
     public String toURI() {
        return toUri(0);
    }

    public static Intent getIntent(String uri) throws URISyntaxException {
        return parseUri(uri, 0);
    }
  ```
  分析源码可以知道，URI和Intent可以相互转换。由Uri可以转成Intent,可以想到由服务端动态下发Intent是可行的
  > Intent的toUri()转化只支持基本对象，不支持序列化对象

- 存取值的底层实现
  Intent的存取值其实就是对其内部mExtras对象的操作，mExtras是一个Bundle，而Bundle对象的实际数据结构是Map（ArrayMap）
  可以理解为Inteng就是ArrayMap的封装

- 显示和隐式Intent
  在activityThread.performLaunchActivity（）中，首先通过反射创建目标Activity对象，然后调activity.attach()方法，继而给
  activity.mIntent赋值，也就是我们开发中getIntent()得到的对象。再往后的流程就是oncreate,所以getIntent()可以在onCreate（）之前获取到值

- clipData传值
  在启动activity时，Intent会进行clipData的数据操作，Api16提供的另一种传递数据的方式
  ```java
  new Intent().setClipData();
  ```

## 序列化方案



