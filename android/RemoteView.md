## RemoteView   
- 介绍   
  android中用于进程间通信的一种view,主要用于桌面小部件和通知消息。
  
- 通信原理   
  RemoteView实现了paracelable接口，可以通过Binder机制在不同进程间传递。当系统进程（桌面小部件和通知显示在系统进程中执行）获取到app传递的
  remoteView对象时，调用其apply()方法。apply中的performApply()方法逐一调用remoteView中action的apply方法进行渲染赋值，然后把remoteView
  填充到对应的位置。
  
- 使用
  - 通知消息
    通过NotificationManager管理。当notificaManager.notify发布一条消息notification,NotificationManger通过Binder和SystemServe通信。当然
    这是内部机制，我们开发不涉及这个内容，只需要通过remoteView来构建notification对象，然后notify这个notification对象即可。
  - 小部件
    通过AppWidgetManager管理。小部件其实也是广播，因为我们编写自己的小部件时继承的AppWidgetProvider类继承子BroadCastReceiver。
    - 编写小部件布局
    - 在res/xml下创建小部件信息xml文件
      ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
          android:initialLayout="@layout/widget"
          android:minHeight="40dp"
          android:minWidth="40dp"
          android:updatePeriodMillis="86400000">
         </appwidget-provider>
       ```
    - 编写小部件类，主要实现其中的onReceive()和onUpdate方法。
      ```java
        class MyAppWidget extends AppWidgetProvider{
          ...
        }
      ```
    - 在AndroidManifest.xml文件中声明小部件
      ```xml
      <receiver android:name=".view.broadcast.MyAppWidget">
            <meta-data android:name="android.appwidget.provider" android:resource="@xml/appwidget_info">
            </meta-data>
            <intent-filter>
                <action android:name="com.eastcom.harup.broadcast.appwidget.CLICK"/>
                <action android:name="android.appwidget.action.APPWIDGET_UPDATE"/>
            </intent-filter>
        </receiver>
      ```
