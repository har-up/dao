### MediaPlayer状态
  - Idel MediaPlayer创建实例并调用reset后处于Idel状态
  - End 创建的MeidiaPlayer实例调用release后变成End状态
  - Error 出错，可以设置setOnErrorLitener监听出错
  - Initialize 处于Idel状态的实例调用setDataSource()后处于Initialize状态（如果在非Idel状态下调用会出错）
  - Prepared 处于Initalize状态的实例调用prepare或prepareAsync后会转到Prepared状态
  - Started 处于Prepare状态的实例在设置一些音量、循环等属性后调用start成功后处于Started状态
  - Paused 播放暂停时所处状态
  - Stopped 调用stop后处于Stoped状态，处于该状态时将不能控制视频播放，直到重新调用prepare进入到Prepared状态
  - PlaybackCompleted 播放完成
  
### SurfaceTexture/SurfaceView/Surface/SurfaceHolder
  - SurfaceTexture
    Android3.0加入的一个类，和SurfaceView类似，不同的是它在接收到图像流之后不需要显示出来，因此可以通过这个类接受解码出来的图像流
    ，对它进行加工。
  - Surface
    处理被屏幕排序的原生Buffer,其内部会创建Canvas，用于绘制
  - SurfaceView
    View的子类，内嵌一个Surface，用于控制图像流展示位置和尺寸
  - SurfaceHolder
    一个接口，可以理解成Surface的监听器
