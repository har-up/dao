# APP的网络框架搭建

## 使用OKHttp搭建自己的网络框架

###OKHttp
  - 介绍
    OKHttp是一款高效的HTTP客户端，其核心主要有路由、连接协议、拦截器、代理、安全性认证、连接池以及网络适配，拦截器主     要是指添加，移除或者转换请求或者回应的头部信息
  - 优点
    -  支持连接同一地址的链接共享同一个socket
    -  通过连接池来减小响应延迟，
    -  透明的GZIP压缩
    -  请求缓存等优势，
  - 缺点
    - 消息返回再子线程中，所以需要自己封装转到主线程更新UI.
### 一、为项目添加jar包或依赖

- 在android studio或idea开发环境下只需在app 的**build.gradle**文件里的**dependencies**中添加依赖项

  ```xml
  dependencies {
      ...
      implementation 'com.squareup.okhttp3:okhttp:3.12.0'
  	implementation 'com.squareup.okio:okio:1.15.0'
  }
  ```

- 如果想以jar包的方式引入依赖项则把上面所说的两个jar包放入main目录下libs文件中（如果没有自己新建一个文件夹名为libs），然后点击鼠标有键选择 “add as Library”。



### 二、编写Http的工具类

> 现有的HttpUtil类在这里，有需要自行更改

- 新建HttpUtil类且该类应为单例模式，以免出现无法预期线程问题。

  ```java
  	//全局变量
  	private static HttpUtil instance;
      private OkHttpClient okHttpClient;
      private Handler handler;
  
      @SuppressLint("NewApi")
      private HttpUtil() {
  
          okHttpClient = new OkHttpClient();
          OkHttpClient.Builder builder = okHttpClient.newBuilder();
          builder.connectTimeout(60, TimeUnit.SECONDS);
          builder.readTimeout(60, TimeUnit.SECONDS);
          builder.callTimeout(60,TimeUnit.SECONDS);
          builder.writeTimeout(60, TimeUnit.SECONDS);
          okHttpClient = builder.build();
          handler = new Handler(Looper.getMainLooper());
      }
  
      /**
       * 通过单例模式构造对象
       * @return HttpUtil
       */
      private static HttpUtil getInstance() {
          if (instance == null) {
              synchronized (HttpUtil.class) {
                  if(instance == null) {
                      instance = new HttpUtil();
                  }
              }
  
          }
          return instance;
      }
  
  ```

  

- 编写通用的post、get及文件上传请求

  > 以下面的post请求为例，给的参数params为JSONObject对象，可以更具自己的项目需求自行更改（请求头的添加同理）。

  ```java
      /**
       * post请求
       * @param url       请求url
       * @param callback  请求回调
       * @param params    请求参数
       */
      @SuppressWarnings("rawtypes")
      public static void post(String url, final ResultCallback callback, JSONObject params) {
          getInstance().postRequest(url, callback, params);
      }
  
      private void postRequest(String url, final ResultCallback callback, JSONObject 			params) {
          Request request;
          String s = "";
          if (params != null){
              s = params.toString();
          }
          request = buildPostRequest(url,s);
          call(callback, request);
      }
  ```



### 三、编写项目接口访问类

- 新建HttpUrl类，类中保存业务接口的url,且封装业务接口的访问

  > 以登陆接口为例

  ```java
  	//后台环境 URL
      private static String URL_PRE = "http://";
      private static String HOST = "dev.eastcom-sw.com/";
      private static String BASE_URL = URL_PRE + HOST + "nrid-open-rest/";
  	 /**
       * 登陆
       *
       * @param loginId
       * @param password
       */
      private static String URL_LOGIN = BASE_URL + "api/auth/login";
  
      public static void login(String loginId, String password, final RequestListner 	listener) {
          // RequestListener为自己应对项目编写的接口，他的传参为ResultModel类。是后台返回的数据统		  格式的规范
          HttpUtil.post(URL_LOGIN, new HttpUtil.ResultCallback() {
              @Override
              public void onSuccess(Object response) {
                  //这里有用到google的json 解析库Gson。记得添加依赖
                  ResultModel resultModel = new Gson().fromJson(response.toString(), 					ResultModel.class);
                  if(listener)
                  listner.onSuccess(resultModel);
              }
  
              @Override
              public void onFailure(Exception e) {
                  listener.onFail(new ErrorModel(-8, e.toString()));
              }
          }, params);
      }
  ```

- 封装获取列表请求的类

  > 按照前边的进行封装，可以得到resultModel对象，但很多接口返回的数据转成resultModel后有其他一些业务实体类的list列表。就这样给业务代码使用，那每个有列表数据的接口都要一一提出来进行处理，这样就会有很多重复的代码以及增大工作量。所以我们想要封装一个获取list列表的类，封装后我们只需要给定list列表的实体类型，封装的类帮我们完成转换工作，在业务层只管取出来用就好了。以下为例：

  - 新建一个类文件，在这里我取名为TypeRequest

  - 编写获取列表数据的方法

    ```java
     	/**
         * 请求-- 获取列表数据
         * @param url
         * @param params
         * @param listner
         */
        public void getListsRequest (String url, JSONObject params, final 		HttpUrl.RequestListner listner, final Class clazz){
            HttpUtil.post(url, new HttpUtil.ResultCallback() {
                @Override
                public void onSuccess(Object response) {
                    List<T> lists = new ArrayList<>();
                    JSONArray list;
                    try {
                        JSONObject jsonObject = new JSONObject(response.toString());
                        list = jsonObject.getJSONArray("list");
                        for (int i = 0; i < list.length(); i++) {
                            String o1 = list.get(i).toString();
                            Gson gson = new GsonBuilder().setDateFormat("yyyy-MM-dd HH:mm:ss").create();
                            T t = (T)gson.fromJson(o1, clazz); //列表数据转换关键代码
                            lists.add(t);
                        }
                    } catch (JSONException e) {
                        e.printStackTrace();
                    }
                    Type type = new TypeToken<ResultModel<T>>(){}.getType();
                    ResultModel resultModel = new 			Gson().fromJson(response.toString().trim(), type);
                    resultModel.setList(lists);
                    if (resultModel.getCode() == 0){
                        listner.onSuccess(resultModel);
                    }else{
                        listner.onFail(new ErrorModel(resultModel.getCode()));
                    }
                }
    
                @Override
                public void onFailure(Exception e) {
    
                }
            },getHeader(),params);
        }
    ```

  - 在HttpUrl编写相关的方法

    ```java
        /**
         *  核查列表
         */
        private static String URL_CHECK_LIST = BASE_URL + "api/check/list";
        public static void getCheckList(int pageSize, int pageIndex, final RequestListner listner,Class clazz){
            JSONObject params = getBaseParam(pageSize, pageIndex);
            new TypeRequest<CheckModel>().getListsRequest(URL_CHECK_LIST, params, new RequestListner() {
                @Override
                public void onSuccess(ResultModel resultModel) {
                    listner.onSuccess(resultModel); //现在，在回调中我们可以直接通过		  resultModel.list来获取实体的列表
                }
                @Override
                public void onFail(ErrorModel errorModel) {
                    listner.onFail(errorModel);
                }
            },clazz);
        }
    ```

    

### 四、Token认证权限的嵌入

> 为了保证用户网络数据的安全性，一般都会为每个请求增加携带**access-token**信息，所以需要在我们的网络框架中加入这一关键技术

- 缓存token信息

  在登陆成功后，我们需要对后台返回的token细心进行缓存。由于app特性，我们只缓存在类中就可以了。在HttpUrl中申明**token**静态变量

  ```java
      public static String TOKEN = "";
  ```

  在登陆成功后，解析得到token信息进行缓存

  ```java
  HttpUtil.post(URL_LOGIN, new HttpUtil.ResultCallback() {
      @Override
      public void onSuccess(Object response) {
          ResultModel resultModel = new Gson().fromJson(response.toString(), ResultModel.class);
          if (resultModel.getCode() == 0) {
              //获取token
              try {
                  JSONObject jsonObject = new JSONObject(resultModel.getItem().toString());
                  TOKEN = jsonObject.getString("token");  //登陆后TOKEN赋予有效值
              } catch (JSONException e) {
                  e.printStackTrace();
              }
              listner.onSuccess(resultModel);
          } else {
              listner.onFail(new ErrorModel(resultModel.getCode()));
          }
      }
  ```

- 在每个请求中添加token请求头

  >  为每一个请求都添加token请求头那不是又有很多重复性的代码？ 我们可以把增加请求头的操作放在TypeRequest里，以达到重用而不是重写的目的

  

### 五、登陆密码的RSA加密处理

​	密码加密与后台的协议规定一般是这样的

​	1、后台返回rsa公钥（格式:**base64位编码的16进制字符串**）

​	2、使用rsa公钥加密密码后传到后台验证（格式：**rsa公钥加密密文,并进行base64位编码**）

​	
