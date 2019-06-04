# WorkManager

## 介绍

是一个Android后台任务调度管理的库，灵活、兼容、简单、可靠。



## 特点

- 支持一次性或周期任务的异步执行

- 支持电量、网络等手机外界因素约束

- 支持work的同步，串行执行

- 兼容到api14

  

## 集成到app

### 基本使用

- **添加依赖库**

  app/build.gradle

  ```xml
  implementation "androidx.work:work-runtime-ktx:$versions.work"
  versions.work = "2.0.1"
  ```

- **创建WorkRequest**

  - 新建BlurWork 继承 Worker
  - 重写doWork（）方法，该方法中的代码就是需要后台执行的任务

- **ViewModel中获取WorkManager实例**

  ```kotlin
  val workManager = WorkManager.getInstance()
  ```

- **把WorkRequest交与WorkManager实例**

    通过WorkManager实例把一个WorkRequest压入队列

  其中有两个方法

  - **OneTimeWorkRequest** 一次性任务
  - **PeriodicWorkRequest**  周期性任务

  ```kotlin
  workManager.enqueue(OneTimeWorkRequest.from(BlurWorker::class.java))
  ```

- **按钮监听执行上一步步骤**

- **数据传递**

  在某些情况下还需要传递数据给**BlurWork**

  - 创建Data对象data

    ```kotlin
    private fun createInputDataForUri(): Data {
        val builder = Data.Builder()
        imageUri?.let {
            builder.putString(KEY_IMAGE_URI, imageUri.toString())
        }
        return builder.build()
    }
    ```

  - 创建带数据的workRequest

    ```kotlin
    val data = createInputDataForUri()
    val workRequest = OneTimeWorkRequest.Builder(BlurWorker::class.java).setInputData(data).build()
    ```

  - WorkManager实例使workRequest入列

    ```kotlin
    workManager.enqueue(workRequest)
    ```

  - BlurWork的doWork（）中获取传递的数据

    ```kotlin
    inputData.getString(KEY_IMAGE_URI)
    ```

### **工作任务链**

- 创建多个Worker。 CleanupWorker和SaveImageToFileWorker,重写doWork方法。

- 创建work链

  先由一个最开始的worker创建WorkContinuation对象。之后可以通过then不断的

  添加worker。

  ```kotlin
  var continuation = workManager.beginWith(OneTimeWorkRequest.from(CleanupWorker::class.java))
  var data = createInputDataForUri()
  var workRequest = OneTimeWorkRequest.Builder(BlurWorker::class.java).setInputData(data).build()
  continuation.then(workRequest)
          .then(OneTimeWorkRequest.from(SaveImageToFileWorker::class.java))
  continuation.enqueue()
  ```

- **确保任务的单一**

  有时有可能某个worker还没完成，但又要开始新的同一类的worker。这时由于两个worker是同一类型的任务，所以需要特点的策略来处理。

  **beginUniqueWork（）**代替**beginWith（）**，其多了两个参数



### 获取工作状态

​	可以获取任何workRequest执行的状态

- 创建workRequest添加tag标志

  ```kotlin
  val request = OneTimeWorkRequestBuilder<SaveImageToFileWorker>().addTag(TAG_OUTPUT).build()
  ```

- 获取WorkInfo

   在viewmodel中声明一个LiveData<WorkInfo>变量且赋值。通过workManager和tag来找到对应的workInfos

  ```
  internal val outputWorkInfos: LiveData<List<WorkInfo>>
  //In the BlurViewModel constructor
  init {
      outputWorkInfos = workManager.getWorkInfosByTagLiveData(TAG_OUTPUT)
  }
  ```

- 观察者观察workInfos的变化

  ```kotlin
  viewModel.outputWorkInfos.observe(this, Observer {
              if (it.isNullOrEmpty()) return@Observer
              for (workInfo in it){
                  if (workInfo.state.isFinished){
                      showWorkFinished()
                  }else{
                      showWorkInProgress()
                  }
              }
          })
  ```

  

### 获取输出数据

- viewmode中新建一个Uri变量

  ```kotlin
  // New instance variable for the WorkInfo
  internal var outputUri: Uri? = null
  internal fun setOutputUri(outputImageUri: String?) {
      outputUri = uriOrNull(outputImageUri)
  }
  ```

- 在获取的workInof中获取信息

  这里的key与inputData中data的key对应

  ```kotlin
  val outputUri = workInfo.outputData.getString(KEY_IMAGE_URI)
  ```



### 中断Work

- 根据名称找到对应work并中断

  这里的名称对应与beginUniqueWork中的名称，即第一个参数

  ```kotlin
   workManager.cancelUniqueWork(IMAGE_MANIPULATION_WORK_NAME)
  ```



### 约束

- 创建约束对象

  ```kotlin
  val constraints = Constraints.Builder().setRequiresCharging(true).build()  //充电约束
  ```

- 赋值给workRequest

  ```kotlin
  OneTimeWorkRequest.Builder(BlurWorker::class.java)
          .setInputData(data)
          .setConstraints(constraints)
          .build()
  ```