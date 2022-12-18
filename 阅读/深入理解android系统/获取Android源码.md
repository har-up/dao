## 获取Android源码
### Linux系统获取
- 下载repo
  创建bin文件夹，并把路径设置到环境变量中
  ```shell
  mkdir ~/bin
  PATH=~/bin:$PATH
  ##下载repo
  curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > ~/bin/repo
  ##设置可执行权限
  chmod a+x ~/bin/repo
  ```

- 初始化repo客户端
  ```shell
  ##创建一个空目录
  mkdir AndroidCode
  cd AndroidCode
  ##下载源码
  ##并未正式发布版本的最新源码
  repo init -u https://android.googlesource.com/platform/manifest
  ##正式发布版本的源码
  repo init -u https://android.googlesource.com/platform/manifest -b android-5.0.0_r1
  ```
  下载过程中需要填写name和email，填写完毕选择Y确认，最后提示repo初始化完成，这时候可以开始同步Android源码了，执行开始同步代码：
  ```shell
  repo sync
  ```

  ### 在Window平台获取Android源码
  现在Windwo平台上搭建一个Linux环境，可以使用cygwin工具。
  cygwin的作用是构建一个Windows上的Linux模拟环境
  下载地址[cygwin](http://cygwin.com/install.html)
  install from internet -> path -> Direc Connection -> add url:http://cygwin.uib.no -> next -> curl search -> install -> next