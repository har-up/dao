Vue开发入门

## 基础

- 了解vue是什么及学洗vue语法基础，请移步[https://cn.vuejs.org/v2/guide/index.html](https://cn.vuejs.org/v2/guide/index.html)。



## 单文件组件

 在学习了基础后，我们了解到可以使用 `Vue.component` 来定义全局组件，紧接着用 `new Vue({ el: '#container'})` 在每个页面内指定一个容器元素来引用我们的组件。这样在比较复杂的项目中会显露出众多的缺点。

我们可以用.vue为后缀单文件来避免这些缺点。

**一个.vue文件中包含三大块：**

- <template>:在这个块里边编写html代码
  
  </template>

- <script>:在这个块里边编写javascript代码
  
  </script>

- <style>:在这个块里边编写样式（css）代码

### 复用单文件组件

> 编写好单文件后，如需在其他文件中用到这个组件有如下几个步骤：

- **导入（在script块中）**

  ```javascript
  import CustmerOrder from '@/components/CustmerOrder'
  ```

- **声明**

  ```javascript
      export default {
          components:{
              CustmerOrder
          }
      }
  ```

- **引用(注意组件名的对应关系)**

  ```javascript
  <custmer-order/>
  ```


## 单页面路由

大多数单页面应用，官方推荐使用支持的 [vue-router 库](https://github.com/vuejs/vue-router),weex项目中非单页面可尝试使用内置的模块**Navigator**

**vue-router的使用有如下步骤：**

- 安装vue-router

  ```bash
  npm install vue-router
  ```

- **安装路由**

  在src目录下新建一个router.js作为全局javascript文件

  ```javascript
  import Router from 'vue-router'
  Vue.use(VueRouter)
  ```

- **注册组件**

  - *在这个router.js中导入需要复用的组件*

  ```javascript
  import MyComponent from '@/components/MyComponent'
  ```

  - *全局路由实例中注册*

    ```javascript
    module.exports = new Router({
      routes: [
        { path: '/go', name: 'my-component',component: MyComponent
        }
      ]
    })
    ```

  - *在入口文件中整合*

    ```javascript
    const router = require('./router');
    new Vue(Vue.util.extend({el: '#root', router}, App));
    ```



- **路由显示**

  - *在.vue文件中的<template>块中放入<router-view>块用于显示*

    ```html
    <div>
         <router-view></router-view>
    </div>
    ```


  - *在适当的时机执行路由*

    ```javascript
    this.$router.push('/go')
    ```

## 网络请求

网络请求框架有**axios**、**resource**等等，但这些需要浏览器内置对象。所以需要使用weex内置的模块**stream**

***使用方法如下：***

- **导入stream**

```javascript
var stream = weex.requireModule('stream')
```

- **fetch请求**

  stream的fetch()方法发起网络请求，示例：

  ```javascript
  stream.fetch({
                      method: 'GET',
                      url: "http://rap2api.taobao.org/app/mock/119086/example/1543298316156?tradeno=''",
                      type:'json'
                  },function(ret) { //请求成功回调
                      if(!ret.ok) {
                          modal.toast({
                              message: ret.message,
                              duration: 5
                          })
                      }else{
                          aa(ret.data.seller)
                          modal.toast({
                              message: "sdsdffse",
                              duration: 10
                          })
                      }
                  }, function(response) {
                      modal.toast({
                          message: response.toString(),
                          duration: 1
                      })
                  })
  ```

  fetch方法具体参数移步http://weex-project.io/cn/references/modules/stream.html



> **注意：在请求到数据后一般都会改变$data值，再进行刷新的操作。其中有几个点这里特别提醒下**

1. 标签为<p>、<div>的文本显示不会对$data值的改变而进行刷新。对应的<text>可以达到所需效果

2. 网络请求的回调里不能直接访问$data数据，所以需要一些处理。在这个我提供一个解决思路，步骤如下：

   - *methods/computed中编写一个改值的方法，如；*

     ```javascript
     change(newValue){
                   this.value = newValue
                 }
     ```

   - *把这个方法的引用赋予请求方法中的一个变量*

     ```javascript
     var aa = this.change
     ```

   - *在网络请求回调中调用aa函数*

     ```javascript
     aa(ret.data)  //ret.data为网络请求回调的数据
     ```


## Vuex的使用

[Vuex](https://vuex.vuejs.org/zh/)是专门为vue.js开发的一套状态管理模式。vue项目加入Vue可以上官网看教程。

在**weex**项目中加入**Vuex**会有些不同，步骤如下：

- **安装**

  ```bash
  npm install vuex --save
  ```

- **导入Vue**

  - *在src目录下新建store.js文件。*

  ```javascript
  import Vuex from 'vuex'
  Vue.use(Vuex);
  module.exports =  new Vuex.Store({
      state: {
          data:""
      },
      mutations: {
      },
      actions:{
      }
  })
  ```

  - 在入口文件（entry.js）中整合。

    ```javascript
    const store = require('./store')
    new Vue(Vue.util.extend({el: '#root',router,store}, App));
    ```

- 数据引用

  ```javascript
  {{$store.state.data}}
  ```

> 注意：加了store.js文件后可能在weex run android的时候会报错：temp目录找不到对应的文件。有两种方法可以解决：
>
> 1. 直接拷贝对应文件到temp目录下
>
> 2. - 在config文件下添加变量
>
>      ![1543550243394](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1543550243394.png)
>
>    - webpack.common.conf.js下添加
>
>      ![1543550313383](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1543550313383.png)

其他[Vuex](https://vuex.vuejs.org/zh/)的特性前往官网学习。