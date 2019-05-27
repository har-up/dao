# Room with a View

## 介绍

View展示数据，官方推荐的架构组件有LiveData、ViewModel、Rom。如下图

![](https://github.com/hedaoqua/dao/blob/master/1558323546381.png?raw=true)

此文档帮你建立一个简单而足够复杂的app，可以作为架构组件的模板。

该app如下功能：

- 使用数据库存储和获取数据，与填充一些words
- 在MainActivity的RecyclerView中显示所有的words
- 点击悬浮按钮时打开另一个activity，当填入word并点击添加时，把该word保存到数据库并显示



## 配置环境

- **新建 app**（**勾选*Use AndroidX artifacts***）

- **导入依赖的组件库**

   	app/build.gradle

  ```xml
  apply plugin: 'kotlin-kapt'
  // Room components
  implementation "androidx.room:room-runtime:$rootProject.roomVersion"
  annotationProcessor "androidx.room:room-compiler:$rootProject.roomVersion"
  androidTestImplementation "androidx.room:room-testing:$rootProject.roomVersion"
  
  // Lifecycle components
  implementation "androidx.lifecycle:lifecycle-extensions:$rootProject.archLifecycleVersion"
  kapt "androidx.room:room-compiler:$rootProject.roomVersion"
  ```
```
  
​	application/build.gradle
  
  ```xml
  ext {
     roomVersion = '2.1.0-alpha06'
     archLifecycleVersion = '2.0.0'
  }
```

  

## 实体类

- **创建实体类**

  为了让Room能够实例实体，需要构造方法和属性的getter（）方法。

- **Room注释**

  - @Entity(tableName="")  注释在实体类上
  - @ColumnInfo(name="") 注释在属性上，数据库表的列名
  - @PrimaryKey 注释在属性上，申明为主键。属性类型为int可以设置自动生成@PrimaryKey(autoGenerate =true)



## Dao层

​	dao,访问数据库的对象。在Room下，一般的表数据操作我们只需要添加注释不需要编写sql语句。

​	下面我们来创建WordDao

 - **新建接口或抽象类**

   添加数据操作方法insert()/delete()/update()/query()

 - **Room注释**

   - @Dao  注释在Dao的类上，必须是接口或抽象类

   - @Insert/@Delete/@Update()/@Query()，注释在对应的方法上

     其中的@Query()需要自己给查询语句

   
   ```java
   @Dao
   interface WordDao {
   
       @Query("SELECT * from word_table ORDER BY word ASC")
       fun getAllWords(): List<Word>
   
       @Insert
       suspend fun insert(word: Word)
   
       @Query("DELETE FROM word_table")
       fun deleteAll()
   }
   ```
   
   > 当insert数据时可能有冲突，可以给定提供一个冲突策略
   >
   > 例如@Insert(onConflict = OnConflictStrategy.REPLACE)





## LiveData

**LiveData**是一个**lifecycle**库中的一个用于数据观察的类。Dao层的方法使用LiveData类型的数据作为返回值，	**Room**库会自动为我们我生成必要的代码—当数据库中的值发生改变时更新LiveData，更新UI。

示例如下:
我们将返回值用LiveData包裹起来，之后在Activity中设置观察者监听数据的改变。onChange方法就是当数据改变时的回调。

```java
// @Query("select * from word_table")
// fun query():List<Word>
   
    @Query("select * from word_table")
    fun query():LiveData<List<Word>>
```

> 如果LiveData独立使用，则需要自己管理数据更新。LiveData没有可用的公开方法去更新存储的数据，需要MutableLiveData替代LiveData, MutableLiveData有两个公开方法setValue(T),postValue(T)。通常，我们会在ViewModel中使用MutableLiveData,且ViewModel只暴露不可变的LiveData类型数据提供访问、观察。



## Room Database

**Room Database**是建立在**Slqite**数据库上的数据库层。它的使用更方便，简单，代替SQLiteOpenHelper。

**Room**有如下特点

	- 使用Dao向数据库发出查询，ORM映射。
	- 为了UI性能，需要时自动在后台线程异步查询
	- 提供编译时检查
	- Room Class 必须继承RoomDatabase 且为抽象类
	- 一个app只需要一个Room实例（内存消耗大）



**整合Room流程**

- 创建一个抽象类继承**RoomDatabase** 

- 注释该类为Room databse,并指定对应的实体类和版本号

- 建立Dao与数据库的关联，相当于为Dao提供一个getter

  代码如下

  ```kotlin
  @Database(entities = [Word::class], version = 1)
  abstract class WordRoomDatabase: RoomDatabase() {
      abstract fun wordDao():WordDao
  }
  ```

- 单例模式

  获取实例时通过Room.databaseBuilder()来完成

  ```kotlin
   companion object {
          private var INSTANCE:WordRoomDatabase? = null
          fun getDatabase(context: Context):WordRoomDatabase{
              return INSTANCE ?: synchronized(this){
                  val instnce = Room.databaseBuilder(context,WordRoomDatabase::class.java,"word_databse")
                      .build()
                  INSTANCE = instnce
                  instnce
              }
          }
      }
  ```

> 当修改数据库的结构时，如果是正式环境，你必须要考虑到数据迁移。See [Understanding migrations with Room](https://zjcqoo.github.io/-----https://medium.com/google-developers/understanding-migrations-with-room-f01e04b07929)



## Repository

仓库类抽象了对多数据源的访问，虽然不属于架构组件库，但是是一个很好的实现架构分离的做法。

仓库类管理查询，允许你使用多种数据源。在常见的示例中，仓库类实现了决定从网络获取数据还是使用缓存在本地数据库中数据的逻辑。

相当与后台架构中的Service层

**Repository层添加流程**

- 新建类，例如WordRepository

- 声明Dao变量和其他数据成员变量

- 构造器中初始化成员变量并获取数据库实例

- 变量getter方法

  Room所有的查询执行都是在单独的线程，LiveData发生改变时会通知观察者observer.

- 数据库访问的方法。包裹insert()

  此类代码不能在UI线程执行，否则会crash

  ```kotlin
  class WordRepository(private val wordDao:WordDao){
  
      val allWords:LiveData<List<Word>> = wordDao.query()
  
      @WorkerThread
      suspend fun insert(word:Word){
          wordDao.insert(word)
      }
  }
  ```

  > Repository中更多处理方式这个案例[BasicSample](https://zjcqoo.github.io/-----https://github.com/googlesamples/android-architecture-components/tree/master/BasicSample) 

 

## ViewModel

**ViewModel**用于向UI提供数据，并在数据更改时保存下来。它作为UI和Repository交互的中间处理层。我们可以使用ViewModel实现多Fragment共享一套数据。

优点：

- 让我们的代码逻辑更清晰，数据与ui分层，遵循单一职责原则。

- 可以实现多Fragment共享一套数据。

- 让Repository与UI完全分离，且没有与数据库的直接访问，让代码更有可测性。

    

添加ViewModel流程

- 新建class继承ViewModel，并使用application上下午为构造器参数

- 声明私有Repository变量以持有对它的引用

- 声明UI需要的数据以缓存数据

- init化代码块中获取Repository实例

- init化代码块中使用Repository实例给声明的UI需要用到的数据赋值

- 对repository的insert()包一层。我们希望insert()方法在主线程之外被调用，在这里可以使用kotlin的协程

  > 协程需要如下依赖库
  >
  > ```
  > implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.1.0-alpha02"
  > 
  > //coroutines
  > implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.1.1"
  > implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.1.1"
  > ```

代码如下

```kotlin
class WordViewModel(application: Application):ViewModel() {

    private val mRepository:WordRepository
    val allWords:LiveData<List<Word>>
    init {
        mRepository = WordRepository(WordRoomDatabase.getDatabase(application).wordDao())
        allWords = mRepository.allWords
    }

    fun insert(word: Word) = viewModelScope.launch (Dispatchers.IO){
        mRepository.insert(word)
    }
}
```

  

## 编写布局

   - **Floating Action Bar**

     ```xml
         <android.support.design.widget.FloatingActionButton
                 android:id="@+id/fab"
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:layout_gravity="bottom|end"
                 android:layout_margin="@dimen/fab_margin"
                 app:srcCompat="@android:drawable/ic_dialog_email"/>
     ```

     

## RecyclerView显示

​	在MainActivity的布局文件中添加RecyclerView控件。并编写对应的Adapter，ViewHolder。

​	如下

- ```kotlin
  class WordListAdapter internal constructor(context: Context): RecyclerView.Adapter<WordListAdapter.WordViewHolde>() {
      private val inflater: LayoutInflater = LayoutInflater.from(context)
      private var words = emptyList<Word>() // Cached copy of words
  
      override fun onCreateViewHolder(p0: ViewGroup, p1: Int): WordViewHolde {
          return WordViewHolde(inflater.inflate(R.layout.recyclerview_item,p0,false))
      }
      override fun getItemCount() = words.size
  
      override fun onBindViewHolder(wordViewHolde: WordViewHolde, p1: Int) {
          wordViewHolde.wordItemView.text = words[p1].word
      }
  
      internal fun setWords(words:List<Word>){
          this.words = words
          notifyDataSetChanged()
      }
  
      inner class WordViewHolde(itemView: View) : RecyclerView.ViewHolder(itemView) {
          val wordItemView:TextView = itemView.findViewById(R.id.textView)
      }
  }
  ```



## 填充数据

​	我们需要考虑到数据显示的生命周期，在app启动时删除之前的内容，并重新填充数据到数据库。

为此，我们建立数据库的回调监听**RoomDatabase.Callback** 重写它的**onOpen（）**方法，但UI线程并不允许与数据库的交互，所以还需使用到协程。

**我们这样做**

- 在获取数据库时，把CoroutineScope作为参数

  ```kotlin
  getDatabase(context: Context,scope: CoroutineScope)
  ```

- 在通过Room.databaseBuilder build一个数据库前添加数据库创建的回调WordDatabaseCallback(),监听onOpen（）回调方法。（onOpen在数据库open时回调过来，比如app开启时）

  ```kotlin
   private class WordDatabaseCallback(
          private val scope: CoroutineScope       //scope传到这里
      ) : RoomDatabase.Callback() {
          override fun onOpen(db: SupportSQLiteDatabase) {
              super.onOpen(db)
              INSTANCE?.let { database ->
                  scope.launch(Dispatchers.IO) {
                      populateDatabase(database.wordDao())
                  }
              }
          }
  
          fun populateDatabase(wordDao: WordDao) {
              wordDao.deleteAll()
              var word = Word("Hello")
              wordDao.insert(word)
              word = Word("World!")
              wordDao.insert(word)
          }
      }
  ```

  

## 数据贯通

建立UI与数据库的数据交互。当点击回车后保存word到数据库，并使用RecyclerView显示数据库当前的内容。

为了显示数据库当前的内容，需要为ViewModel中的LiveData添加观察者用以观察数据的变更。

当监听到变更时，我们就通过RecyclerView的adapter来赋值setWords()。

- 在MainActivity中申明WordViewModel变量，并在onCreate()中赋值

  ```kotlin
    private lateinit var mWordViewModel: WordViewModel
    ...
    mWordViewModel = ViewModelProviders.of(this).get(WordViewModel::class.java)
  ```

- WordViewModel变量中的LiveData类型数据添加观察者观察数据变更

  ```kotlin
  mWordViewModel.allWords.observe(this, Observer { words ->
              words?.let {
                  wordListAdapter.setWords(words)
              }
          })
  ```

  



## 总结

The components of the app are:

- `MainActivity`: displays words in a list using a `RecyclerView` and the `WordListAdapter`. In the `MainActivity`, there is an `Observer` that observes the words LiveData from the database and is notified when they change.
- `NewWordActivity:` adds a new word to the list.
- `WordViewModel`(*): provides methods for accessing the data layer, and it returns LiveData so that MainActivity can set up the observer relationship.
- `LiveData<List<Word>>`: Makes possible the automatic updates in the UI components. In the `MainActivity`, there is an `Observer` that observes the words LiveData from the database and is notified when they change.
- `Repository:` manages one or more data sources. The `Repository` exposes methods for the ViewModel to interact with the underlying data provider. In this app, that backend is a Room database.
- `Room`: is a wrapper around and implements a SQLite database. Room does a lot of work for you that you used to have to do yourself.
- DAO: maps method calls to database queries, so that when the Repository calls a method such as `getAllWords()`, Room can execute **SELECT \* from word_table ORDER BY word ASC****.**
- `Word`: is the entity class that contains a single work.