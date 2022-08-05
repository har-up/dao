## 源码分析
SharedPreferences实际上是一个xml和i/o操作的集合
SharedPreferences会对磁盘的文件进行操作，但磁盘操作都是比较耗时的，所以Android会将磁盘内容读取到内存中，直接对内存进行操作，这就是SharedPreferences的缓存机制
- 缓存机制
  每次调用context.getSharedPreferences（String name,int mode）其实就是通过name在磁盘上找到自己对应的一个file，然后建立一个流，因为每次执行getSharedPreferences
  就建立一次流的方式过于低效，所以源码中就建立了一个cache
  源码中是用一个map做缓存
  建立SharedPreferences的步骤是涉及到两个缓存系统
  一级缓存是通过name得到本地的file的时候，会对file建立一个缓存
  二级缓存是在通过file得到最终的SharedPreferences的时候，会对SharedPreferences建立一个缓存

- SharedPreferencesImpl
  SharedPreferencesImpl才是SharedPreferences的本体，SharedPreferencesImpl的建立过程就是通过读取xml文件解析为map的过程
  流程如下
  - 通过文件流读取文件，文件内容解析放入map
  - 如果有备灾文件，则用备灾文件覆盖原始文件
  - 因为读磁盘是异步操作，所以要加上一个是否读盘完成的标志位
  - 读取完成后用notifyAll（）方法激活所有等待加载的其他线程
  > 不建议在构造函数中向SharedPreferencesImpl这样做长时间的异步操作，加标志位很容易疏忽出问题

  