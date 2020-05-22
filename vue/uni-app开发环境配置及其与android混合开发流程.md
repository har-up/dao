# uni-app开发环境配置及混合开发流程

## NodeJS开发环境

- ### 安装NVM

  - **下载安装**

    > NVM是一个支持多版本NodeJS的版本管理工具，如果本机已安装了NodeJS建议先卸载。

    - 下载[NodeJS](https://github.com/coreybutler/nvm-windows/releases/download/1.1.7/nvm-setup.zip)

    - 解压后直接运行安装：设置好安装目录及后面NodeJs的安装目录。


  - **配置环境变量**

    > 路径对应你安装时给定的安装目录

    ![1543887190762](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1543887190762.png)

  - **验证安装**

    命令行下执行：

    ```
    nvm –v
    ```

     显示信息如下：

    ![1543887319951](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1543887319951.png)



- ### 安装NodeJS

  > 打开命令行操作窗口进行以下步骤

  -  安装

    ```
    npm install <version> [<arch>]
    ```



    > <version>表示要安装的版本; arch参数表示系统位数，默认是64位，如果是32位操作系统，需要执行命令：nvm install <version> 32
    >
    >
    >
    > 特别说明：
    >
    > l 查看当前有哪些版本：nvm ls-remote （Window版本可能无此命令）
    >
    > l 安装当前最新稳定版本: nvm install stable 或 nvm install stable 32
    >
    > l 查看已安装的版本：nvm ls
    >
    > l 切换Node版本: nvm use <版本>
    >
    >
    >
    > 验证及查看当前安装的node及npm版本:
    >
    > ```
    > node -v 
    > 
    > npm -v
    > ```
    >
    >

  - 安装淘宝镜像(可选)

    ```
    npm install cnpm -g --registry=https://registry.npm.taobao.org
    ```



    >  安装成功后使用***cnpm命令代替npm***

## Git本地环境配置

- **下载安装Git**

  **in64:** <https://github.com/git-for-windows/git/releases/download/v2.19.1.windows.1/Git-2.19.1-64-bit.exe>

  **Win32:** <https://github.com/git-for-windows/git/releases/download/v2.19.1.windows.1/Git-2.19.1-32-bit.exe>

- **Git全局配置**

  ```
  git config –global user.name “<姓名>”
  
  git config –global user.email “<公司邮箱>”
  ```

- **配置本地仓库**

  ```
  git clone http://gitlab.eastcom-sw.com/<组名>/<项目名>.git
  
  cd <项目名>
  ```




## Vue开发环境

- **前提条件**

  已正确安装NodeJS开发环境。

  已正确安装Git开发环境。

- **安装Vue脚手架**

  ```
  npm install -g @vue/cli
  ```

  验证安装：vue -V 

- **Vue项目创建、运行、编译打包**（创建vue项目才走该步骤，创建nui-app项目跳过该步骤）

  - *新建项目*

    ```
    vue create <项目名>  或vue ui (图形接口创建项目)
    ```


  - *下载依赖*

    ```
    cd <项目名>(进入项目根目录)
    
    npm install （下载依赖）
    ```


  - *运行项目*

    ```
    cnpm run serve
    ```

  - *编译打包*

    ```
    npm run build
    ```


## 创建uni-app项目

- **创建项目vue** 

  ```
  create -p dcloudio/uni-preset-vue my-project
  ```



  > 此时，会提示选择项目模板，初次体验建议选择 `hello uni-app` 项目模板
  >
  > ![img](http://img.cdn.aliyun.dcloud.net.cn/guide/uniapp/h5-cli-01.png)
  >
  > 提示：通过该命令创建项目前提是全局安装了vue-cli

- **进入目录并运行**

  ```
  cd my-project
  npm run serve
  ```

  运行成功后，控制台会输出H5网站访问地址

  ![img](http://img.cdn.aliyun.dcloud.net.cn/guide/uniapp/h5-cli-02.png)

  启动chromel浏览器并切换为手机调试模式，访问如上地址即可体验。

## nui-app与原生混合开发

- **插件开发**

  - *准备*

    下载HTML5+基座的Android版SDK[点击下载](http://ask.dcloud.net.cn/article/103)解压后将HBuilder-Integrate工程导入AndroidStudio.

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
      >

  - *关联JS插件名和原生类*

    > 在编写扩展插件时需要修改“/assets/data”中dcloud_properties.xml文件，在其中添加JS对象名称和Android包的类名对应关系，SDK会根据对应的类名查找并生成相应的对象并执行对应的逻辑。

    ![img](https://img-cdn-qiniu.dcloud.net.cn/uploads/article/20141017/5489a76e7be78c295b8c786c77835162.png)

    >
    >
    >在应用的manifest.json文件中还需要添加扩展插件的应用使用权限
    >
    >![img](https://img-cdn-qiniu.dcloud.net.cn/uploads/article/20141017/5aa58c39dd81aba2838bdb60b2509ddb.png)

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

