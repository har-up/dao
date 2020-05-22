# uni-app快速开发模板（微信登陆）

​	一个uni-app开发一般都有登陆、注册等流程；集成vuex状态管理框架、网络请求。所以在此提供一个uni-app快速开发模板。避免从0开始搭建浪费时间，缩减开发周期。

## 模板开发介绍

### 登陆

该模板的登陆流程以微信登陆为基础，在这里说明下微信登陆的流程

- 获取微信权限

  一般进入到我们的app时，需要用户授权用户微信账户信息。只有获取拥有用户的微信账户权限，才能通过微信登陆接口获取用户code

  ```javascript
  wx.getSetting({
  			success: function(res) {
  				if (res.authSetting['scope.userInfo']) {
  					wx.getUserInfo({
  						success: function(res) {
  							context.weixinLogin(); // 调用微信的 wx.login 接口，获取code
  						}
  					});
  				} else {     // 用户还没有授权
  					uni.hideLoading();
  					context.setIsHide(false); // 改变 isHide 的值，显示授权页面
  				}
  			}
  		});
  ```

- 微信登陆获取code

  - 获取到code之后一般流程是传递给后台，后台通过微信接口获取openId，openId就是对应一个用户的唯一标识符。同时，后台返回资源访问token给app,后续的资源访问需要携带这后台返回的token。

  - 当然也可以直接使用微信的提供的接口直接获取 openid，具体的流程就要有所改变了。

  ```javascript
  wx.login({
  				success: res => {
  					// 获取到用户的 code 之后：res.code
  					this.$api.authToken(res.code).then(res => {
  						if (res.data) {
  							res.data.date = new Date()
  							this.setTokenInfo(res.data)
  							let context = this
  							uni.setStorageSync("access_token",res.data.access_token)
  							context.userInfo()
  						}else{
  						}
  					});
  				}
  			});
  ```

### token处理

​	既然资源访问接口都需要携带token，那么就把它封装到请求框架中，避免每个接口都传递token这个参数。可以在请求拦截中添加token。其中还会有token失效的问题，可以在请求前拦截判断token是否失效，如果失效先请求刷新token的接口，这里要考虑到请求同步问题；还有一种方案就是，在请求响应回调中判断响应是偶是token失效，若是token则跳到登陆页面重新登陆（这种方式微信小程序不适用）。

```javascript
http.interceptor.request = (config) => { //请求拦截
	let token = store.state.tokenInfo.access_token
	let tokenInfo = store.state.tokenInfo
	if (token) {
		config.header.Authorization = 'bearer ' + token
	}
	return config
}
```

### 全局状态管理

​	全局状态管理使用的是vuex状态管理框架  ，vuex的使用看https://vuex.vuejs.org/zh/

​	通过官方教程我们知道了在vue文件中怎么使用vuex,但在js文件中需要访问器中的变量该怎么办？

​	在js文件中通过如下方式可以访问到store.js中的数据logined

```javascript
import store from '@/store/store.js';  //导入store
store.state.logined  //访问数据
store.commit('setLogined',true) //修改数据（需要在store的mutations中声明setLogined方法）
```



## 模板搭建流程

- 打开HbuilderX新建一个uni-app项目

- 集成vuex

  uni-app内置了vuex, 所以我们只需要配置一下即可使用

  - 项目目录下新建store/store.js文件

  - 在main.js中注册

    ```javascript
    import store from './store/store'
    const app = new Vue({
    	store,
        ...App
    })
    ```

- 集成网络请求框架

  把封装代码复制到项目目录下，并在main.js中引入

  ```
  const app = new Vue({
  	store,
  	api,
  	util,
      ...App
  })
  ```

- 三方ui框架

  由于uni-app自带的组件不够美观，如果得不到足够的ui支持，一般开发者很难做出漂亮的页面。我在插件市场选了一个三方ui框架引入使用

  - 导入colorui代码到项目目录

  - 在app.vue中引入样式

    ```javascript
    <style>
    	@import 'colorui/main.css';
    	@import 'colorui/icon.css';
    </style>
    
    ```

  