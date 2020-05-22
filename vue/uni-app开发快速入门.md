# uni-app开发快速入门

**该文档用于帮助未接触uni-app开发的同事快速入门进行开发任务。之后的同事如果遇到问题及解决方案或现有技术点有更好的方案可以更新该文档，使其不断完善。**

### 官方文档

​	刚接触到该技术，首先需要学会在官方寻找答案，多数问题都可以在官方网站找到答案。

​	**官方网站**：https://uniapp.dcloud.io/

​	**官方论坛**：https://ask.dcloud.net.cn/explore/

**快速进行开发请首先完成以下步骤**：

- 到官网的**快速体验**下载对应平台的app,下载后可以看到当前uni-app可以可以直接使用的一些组件、接口、模板。
- 官网左侧选项**开源项目汇总**，下载**Hello uni-app**这个项目是上一个步骤中下载的示例app的源码。示例app中的每个页面效果都可已在该项目源码的**pages/**目录下找到。（当然也可以下载其他的项目源码参考一些效果）。
- 若要用到示例app中的功能，可以找到对应的源码阅读，引用到开发的项目中。如果对该功能有什么疑问，可以在官网的搜索框中搜索对应的功能点。比如搜索：*"segmentedControl"*就可以查看对应组件的使用说明；搜索“uploadFile”可以找到uni上传文件的接口说明。
- 自此，基本上就可以逐步进行开发了。如果有遇到什么问题可以在官方论坛中搜索寻找答案。想要的一些功能没有的话可以前往官网的**插件市场**中找一 下。
- 如果在开发过程中遇到某个难点。请记录下来，在找到解决方案后，总结更新到该文档上。

### tabBar(app底部标签)

使用uniapp自带的tabBar只需在pages.json中配置tab。但只支持底部tab，小程序支持顶部tap。

> 当页面是tabBar配置中的一个list时，到该页面就会显示底部tab。

```javascript
"tabBar": {
		"color": "#7a7e83",
		"selectedColor": "#0faeff",
		"backgroundColor": "#ffffff",
		"list": [{
			"pagePath": "pages/icon/test",
			"text": "test",
			"iconPath": "static/img/home.png",
			"selectedIconPath": "static/img/homeHL.png"
		}, {
			"pagePath": "pages/main/main",
			"text": "首页",
			"iconPath": "static/img/home.png",
			"selectedIconPath": "static/img/homeHL.png"
		}, {
			"pagePath": "pages/user/user",
			"text": "我的",
			"iconPath": "static/img/user.png",
			"selectedIconPath": "static/img/userHL.png"
		}]
	}
```



### 页面布局

​	uni-app推荐使用flex(弹性布局)。

​	弹性布局的学习请前往http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html

### 设置为横屏

锁定屏幕方向

void plus.screen.lockOrientation( String orientation );

说明：

锁定屏幕方向后屏幕只能按锁定的屏幕方向显示，关闭当前页面后仍然有效。 可再次调用此方法修改屏幕锁定方向或调用unlockOrientation()方法恢复到应用的默认值。

参数：

orientation: ( String ) 必选 要锁定的屏幕方向值
锁定屏幕方向可取以下值： "portrait-primary": 竖屏正方向； "portrait-secondary": 竖屏反方向，屏幕正方向按顺时针旋转180°； "landscape-primary": 横屏正方向，屏幕正方向按顺时针旋转90°； "landscape-secondary": 横屏方向，屏幕正方向按顺时针旋转270°； "portrait": 竖屏正方向或反方向，根据设备重力感应器自动调整； "landscape": 横屏正方向或反方向，根据设备重力感应器自动调整；

### vuex状态管理（全局数据处理）

> uni中可以使用vuex进行全局状态管理,一些组件间共享的数据或多页面用到的数据可以放在vuex中统一管理。

在vuex状态文件中定义状态，如：

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
	state: {
		/**
		 * 是否需要强制登录
		 */
		forcedLogin: false,
		hasLogin: false,
		userName: "sss",
		token: "",
		billDetail: { //出货单信息
			date: "2018-1-1",
			customerName: "张三",
			goods: [{
				id: "1",
				name: "土豆",
				price: 3.5,
				number: 100,
				account: 350
			}]
		},
		editTempGoods: [{
				id: "1",
				name: "土豆",
				price: 3.5,
				number: 100,
				account: 350
			},
			{
				id:"2",
				name: '白菜',
				price: 2.5,
				number: 100,
				count_price: 250
			}
		],
	},
	mutations: {
		login(state, userName) {
			state.userName = userName || '新用户';
			state.hasLogin = true;
		},
		logout(state) {
			state.userName = "";
			state.hasLogin = false;
		},
		setToken(state, token) {
			state.token = token
		}
	}
})
export default store

```

#### 访问全局数据

> **在组件或页面中需要用到状态，该怎么引用呢？**

- 导入mapState

  ```javascript
  	import {
  		mapState,
  	} from 'vuex';
  ```

- 在计算属性中声明需要用到的状态

  > 有两种方式：
  >
  > - 如果只需要用到vuex状态，这样写：
  >
  >   ```javascript
  >   computed:(['hasLogin','forcedLogin','userName'])
  >   ```
  >
  > - 还需要扩展一些计算属性这样写：
  >
  >   ```javascript
  >   computed: {
  >   			...mapState(['hasLogin','forcedLogin','userName']),
  >   			startDate() {
  >   				return this.getDate('start');
  >   			},
  >   			endDate() {
  >   				return this.getDate('end');
  >   			}
  >   		}
  >   ```

现在在vue的template中可以就直接引用到声明的状态了。

#### 修改全局数据

> **在组件或页面中需要修改状态，该怎么修改呢？**

- 在vuex状态文件的 mutations中定义方法。如

  ```javascript
  	mutations: {
  		login(state, userName) {
  			state.userName = userName || '新用户';
  			state.hasLogin = true;
  		},
  		logout(state) {
  			state.userName = "";
  			state.hasLogin = false;
  		},
  		setToken(state, token) {
  			state.token = token
  		}
  	}
  ```

- 导入mapMutations

  ```javascript
  	import {
  		mapMutations,
  	} from 'vuex';
  ```

- 申明

  在组件或页面中的methods中定义

  ```javascript
  methods: {
  	...mapMutations(['login','logout','setToken']),
  	}
  ```

  现在在vue的template中可以就直接使用方法来修改状态了。



### 程序退出问题

 	问题描述：uni-app自带程序退出提示，没有监听程序退出的方法,它自带的退出提示是在发布页（pages.json中配置的第一个vue页面）生效。程序没登陆的时候首次进入的该是登录页，然而在很多情况下我们需要在主页有退出程序的提示，且在该页面可以按两次后退键直接退出程序。

​	**解决方案**：把主页设为发布页，在发布页的onLoad()方法中判断是否需要进入登陆流程，如果需要登陆则通过API——**uni.reDirect()**跳转到登陆页，登陆成功后通过**uni.reDirect()**跳转到主页。

> *uni.reDirect()接口*：当前页出栈，指定页入栈。

### app本地缓存

[uni.setStorageSync](https://uniapp.dcloud.io/api/storage/storage?id=setstoragesync)：将 data 存储在本地缓存中指定的 key 中。因为vuex不是持久化的状态，用户关掉程序，然后再次启动程序，就会丢失掉用户信息。这里我想到的解决办法就是使用storage，将用户信息保存在本地缓存中。

### uni.request封装

封装uni-app的网络请求，可以避免写一些每个请求都重复的参数，比如token。

在util.js中写编写如下方法

```javascript
function sourceRequest(params){
	uni.request({
		url: 'http://' + params.url,
		method: 'GET',
        header:params.header,
		data: params.data,
		success: res => {
            if(params.success!=undefined){
                params.success(res)
            	}
            },
		fail: (error) => {
            if(params.fail!=undefined){
                params.fail(error)
            	}
        	},
        complete: (msg) => {
            if(params.compelete!=undefined){
                params.complete(msg)
            }
        }
	});
}

module.exports = {  //声明提供访问
	sourceRequest: sourceRequest,
}
```

在main.js中引入util:

```
import util from './common/util.js'
Vue.prototype.util = util
```

使用：

```javascript
this.util.sourceRequest({
					url:"www.baidu.com",
					data:{},
					success:function(res){console.log(res.data)},
					fail:function(error){
						console.log(error);
					}
				})
```





### 动画相关

http://ask.dcloud.net.cn/article/225



### 列表数据的归类

> 需求说明：在app中显示的列表信息（比如购物App中的订单）需要分类显示（比如按日期分类）。

**技术方案：**在列表数据从网络加载后遍历每一条数据。给每条数据加上两个属性，一个属性用于判断是否显示分类信息，另一个属性为所属的分类类别。（也可以与后台协定，让后台返回的每条数据都携带这些信息）。



### nui-app与原生混合开发

> [uni-app官网](https://uniapp.dcloud.io/)

- **插件开发**

  - *准备*

    下载HTML5+基座的Android版SDK[点击下载](http://ask.dcloud.net.cn/article/103)解压后将HBuilder-Integrate工程导入[AndroidStudio](https://developer.android.google.cn/studio/).

  - *创建插件类*

    - 创建一个继承自StandardFeature的类，实现第三方插件扩展。

    - 插件类的***扩展方法***

      > 扩展方法有两个传入参数:
      > ​	IWebview pWebview : 发起请求的webview
      > ​	JSONArray array : JS请求传入的参数
      >
      > 返回值（有同步执行返回和异步执行返回）：
      >
      > - 同步（框架通过此类方法对返回值进行包裹）
      >
      > ```java
      > JSUtil.wrapJsVar("Html5 Plus Plugin Hello1!",true);
      > ```
      >
      >
      >
      > - 异步
      >
      >   ```java
      >   JSUtil.execCallback(pWebview, cbId, (which==AlertDialog.BUTTON_POSITIVE)?"ok":"cancel", JSUtil.OK, false, false); 
      >   ```

  - *关联JS插件名和原生类*

    > 在编写扩展插件时需要修改“/assets/data”中dcloud_properties.xml文件，在其中添加JS对象名称和Android包的类名对应关系，SDK会根据对应的类名查找并生成相应的对象并执行对应的逻辑。

    ![img](https://img-cdn-qiniu.dcloud.net.cn/uploads/article/20141017/5489a76e7be78c295b8c786c77835162.png)

    >
    >
    > 在应用的manifest.json文件中还需要添加扩展插件的应用使用权限
    >
    > ![img](https://img-cdn-qiniu.dcloud.net.cn/uploads/article/20141017/5aa58c39dd81aba2838bdb60b2509ddb.png)

- **uni-app调用插件扩展的方法**

  在uni-app的.vue单页面文件中通过plus.bridge.exec(“插件名”，“扩展方法名”)

  ![1543891995863](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1543891995863.png)

- **离线打包**

  - 安装[HBuilderX](http://www.dcloud.io/hbuilderx.html)

  - [微信开发工具下载](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)

  - 生成**项目的**本地打包app资源

    ![1543892439252](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1543892439252.png)

  - 整合

    - 生成完毕后放到前边插件开发的HBuilder-Integrate工程的/assets/apps 目录下

    - 在/assets/data下的dcloud_control.xml中修改所需整合的uni-app编译成的项目id。

      (appid和本地打包app资源中的manifest.json文件里边的id一致)

      ![1543893092520](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1543893092520.png)

- 打包/运行



