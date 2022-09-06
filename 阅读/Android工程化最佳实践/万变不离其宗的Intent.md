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
- Serializable
- Parcelable
- Protocol Buffer
- Twitter/Serial
> 常见问题
  - 父类的序列化问题
    可以在子类中序列化也可以在父类中序列化，子类中别忘记调用父类的序列化，super.writeToParacel(transient关键字可以避免变量被序列化)
  - 类型转换异常
    若存入的数据类型为子类的数组，则取出的数据类型为父类数组，无法强转为子类
    数组类型是不支持向下强制转型的
  - 重复启动问题
    如果Activity已经在任务栈中，再次调用startActivity，在onResume中获取到的Intent会还是第一次的，不是startActivity传过来的；建议通过onNewIntent回调来
    获取新的Intent,然后调用setIntent方法充值Intent

  - 传递大对象
    Intent在传递数据时是有大小限制的，一般超过1MB就会异常，尽量不要将bitmap对象放进Intent传递。
    解决方案：建立一个存储器
    ```java
    public class Model implements Parcelable {
    
    protected Model(Parcel in) {
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        int i = ModelStorage.getInstance().putModel(this);
        dest.writeInt(i);
    }

    @Override
    public int describeContents() {
        return 0;
    }

    public static final Creator<Model> CREATOR = new Creator<Model>() {
        @Override
        public Model createFromParcel(Parcel in) {
            return ModelStorage.getInstance().getModel(in.readInt());
        }

        @Override
        public Model[] newArray(int size) {
            return new Model[size];
        }
    };
    }
    public class ModelStorage {

    private static ModelStorage mInstance = null;
    private Map<Integer, Model> map = new ArrayMap<>();

    public static ModelStorage getInstance() {
        if (mInstance == null) {
            synchronized (ModelStorage.class) {
                if (mInstance == null) {
                    return new ModelStorage();
                }
            }
        }
        return mInstance;
    }

    private ModelStorage() {
    }

    public int putModel(Model model) {
        if (map.containsValue(model)) {
            for (Map.Entry<Integer, Model> entry : map.entrySet()) {
                if (entry.getValue() == model) {
                    return entry.getKey();
                }
            }
            return 0;
        } else {
            int index = map.size();
            map.put(index, model);
            return index;
        }
    }

    public Model getModel(int key) {
        if (map.size() == 0) {
            return null;
        }
        return map.remove(key);
    }
    }
    ```

## 传值库-Parceler
聚美优品的jumeiRdGroup/parceler
- 降低Key的维护成本
  目前维护Intent的key的很多做法是维护一个统一的类存放Intent的key,这样做存在一些问题
  - 开发者需要维护Key和两个Activity的关系
  - 如果是组件化方案，阿么每个组件需要自定义自己的Key文件
  - 多个Activity 可能有相同的Key，存在重命名一个Key而影响其他Key的逻辑隐患
  解决方案：在Activity中建立私有的Key，提供startActivity的静态方法，提升Activity的内聚性
  
- 自动维护Inteng的key
  当一个Activity中需要传入大量的值时，编写Key和取值将是一个繁琐的过程
  通常情况下Key和变量的name是有对应关系的，那么我们可以使定义变量的时候自动生成Key，这便是注解的功能之一。
  Parceler库提供了注解@Arg,这个注解会做到根据变量名自动生成对应的Key

- Jetpack中的自动化
  Jetpack中的Navigation中提供了类似的自动生成方案
- 自动保存状态
  Parceler提供了onSaveInstanceState()和onRestoreInstanceState()方法的存取方法，用于解决数据在异常情况下保存和恢复的问题
  
- 处理ClassCastException
  Intent会有一些类型转换方面的问题
  - 存入StringBuffer/StringBuilder，得到String
  - 存入Parcelable子类的数组，得到parcelable[]
  - 存入charSequence子类的数组，得到CharSequence[]
  - 存入HashMap的子类，得到HashMap
  针对上述问题，Parceler在取值时调用了wrapCast()方法进行了类型的处理

- IntentLauncher
  Parceler提供的一个简单的工具类，方便跳转调用

- 统一存取的API
  
