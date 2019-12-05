## window创建过程
  ### 获取windowManager   
  在activity中可以获取到WindowManager类的对象windowManager，WindwoManager是一个接口，获取到的其实是一个WindowManagerImpl对象。
  - 在Activity类中有getWindowManger()方法，其内部通过Window对象mWindow获取到windowManager
  - 其中的mWindow是在Activity的attch方法中通过new PhoneWindow()来得到的
  - PhoneWindow继承Window类
  - Window类中的getWidowManager中返回windowManager。windowManager是通过Window类中的setWidowManager来赋值的
  - setWindowManager又是在Activity的attach方法中调用，如下。拿了系统的WindwoService作为WidowManager作为参数入参
  ```java
        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
  ```          
   -  Window的setWindowManager又做了些什么操作呢，以下代码说明返回的windowManger对象就是WindowManagerImpl对象
   ```java
      public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
            boolean hardwareAccelerated) {
        mAppToken = appToken;
        mAppName = appName;
        mHardwareAccelerated = hardwareAccelerated
                || SystemProperties.getBoolean(PROPERTY_HARDWARE_UI, false);
        if (wm == null) {
            wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
        }
        mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
      }
   ```
  ### addView   
  获取到windowManager对象后，可以通过它的addView(view,Window.layoutParams)方法来添加一个Window。
  - WindowManagerImpl中的addView并没有真正添加逻辑，而是通过WindowManagerGlobal对象mGlobal调用它的addView()。mGlobal单例模式，在WindowManagerImpl类全局域赋值
  ```java
        private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
  ```
  - 在WindowManagerGlobal中有三个重要的容器，mViews（存放所有window对应的view）、mRoots（存放所有window对应的ViewRootImpl）、mParams(存放所有window所对应的布局参数)
    
  - 在setView中会将将要操作的window view,params以及viewRootImpl添加到对应的容器。然后掉调用viewRootImpl的setView()方法，调用setView方法中的requestLayout()来完成异步刷新请求。
  - requestLayout() -> scheduleTraversals() -> pokeDrawLockIfNeeded() -> mWindowSession.pokeDrawLock(mWindow)
  - mWindowSession 在ViewRootImpl的构造方法中给出,mWindowSession是IWindowSession类型，是一个Binder对象，真正的实现类是Session，session内部会通过WindowManagerService来实现Window的添加。
    ```java
    public ViewRootImpl(Context context, Display display) {
      ...
      mWindowSession = WindowManagerGlobal.getWindowSession();
      ...
    }
    ```
 
    
