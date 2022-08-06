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

- 值操作
  取值操作就是从map中根据key取值，这里会根据标志mLoaded来判断文件的内容是否已经加载到内存中，没有的话就锁等待；
  设置值的时候不会直接操作mMap，而是作用于mModified对象，该对象用于记录修改的操作，相当于一个记录器，在执行commit的时候，会将mMap的值和mModified进行对比合并再提交
  只读和修改分离是editor设计的重点

- 提交操作
  apply()和commit()
  apply是将修改异步写入磁盘，commit是直接写入磁盘。由于每次都写入，所以commit的效率相对较低，apply用的是单线程的队列操作，在频繁调用时用apply效率较高。
  使用两种中的哪一个取决于场景，如果很重视写入的结果，commit更适合，如果频繁的修改用apply更合适。
  > commit在操作中不一定都是在主线程中进行的，当判断有线程在修改时，会将写入操作放入单线程队列排队执行，此时就在这个单线程中执行而非主线程。
  
## 异常处理
- name为null
  在4.4之前如果name为null，建立SharedPreferences的文件名为null，强烈建议传入值非空，为了方便合作的使用整理和防止冲突，可以将name设置为静态变量放在一个接口中

- 管理好key的取名
  避免key为null的情况
  > 推荐一个SharedPreferences对应一个自己的Key类，存放对应SharedPreferences中的所有key,利用java变量不重名的语法检查可以很好的避免key冲突的问题。

- 清空操作失败
  要把clear()操作当做原子操作，调用clear后立即调用commit()或apply()

- 磁盘写入异常
  写入流程，自动备份之前的file，备份为后缀名为.bak的文件，得到输入流，开始写盘。写入完成后，返回true，将之前的备份文件删除。
  如果出现异常，那么.bak文件就会保留，如果实际开发中想要判断写入的失败率，可以遍历一下SharedPreferences文件中国的.bak文件。

- 出现ANR
  由于涉及到I/O操作，在偶然情况下回发生ANR异常。在之前的介绍中，有涉及到锁等待的代码，Preferences出现ANR大概率是由于这个锁等待的问题，由于
  锁一直没开，即耗时的i/o操作一直没有执行完成。
  一个场景：执行SharedPreferences的写盘操作后，立刻启动一个service，这是Android会强制等待SharedPreferences的写操作完成，将异步变成同步，如果写盘数据量大
 就很容易产生ANR。
 
- 存序列化对象
  更新版本后读取之前持久化的对象出错；确保序列化对象的SerialVersionUID有定义

- 多App和多进程访问异常
  Android N开始，系统不在支持App访问另一个App中的SharedPreferences

## 性能优化
- 避免存储大量数据
- 尽可能提前初始化
  可以将当前界面需要建立SharedPreferences的时机放在activity.onCreate中或Fragment.onCreate中，这样就会提前加载到内存中，建立好缓存

- 避免Key过长
  如果存放的Key过长，计算key的hashcode的时间就会加长，影响效率
  SharedPreferences中用到的数据接口很有意思，可以研究一下
  - ContextImpl中的sSharedPrefsCache是一个arrayMap
  - 缓存SharedPreferencesImpl的cache是一个arrayMap
  - SharedPreferencesImpl内部的mMap是一个HashMap
  在Android中，ArrayMap是特有的数据接口，当数据有如下特征时可以选择ArrayMap
  - 数据量小（< 1000）,采用二分查找，速度快
  - 数据量小，无需考虑扩容
  - 运行期间不执行或较少执行remove操作
  - 数据的插入操作十分低频
  - 本身包含子map对象，即map中存在map的结构
  
- 多次操作，批量提交
  官方推荐多次修改完成后，一次性提交。

- 缓存Editor对象
  每次调用edit()就会new一个Editor对象，在封装类中可以将editor变成类的成员变量

- 不存放Html和Json
  Html和Json在处理时会有很多转义字符，在解析时会有额外的效率消耗

- 拆分高频和低频操作

## 封装SharedPreferences
- PreferenceDataStore
  官方的代替类，只是提供了更高的抽象层，可以用它封装现有的SharedPreferences,以便以后进行底层的替换

- 三方库Treasure
  - 注解+接口
  - 多用户存储设计
  - 统一管理Key
  - 自动判断返回值类型
  - 定义使用apply还是commit提交
  - 存放序列化对象
  - 支持数据格式转换器
  -
