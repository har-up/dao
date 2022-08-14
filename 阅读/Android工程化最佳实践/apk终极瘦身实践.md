## 安装包的构成
android studio的Build-Analyze APK 可以直接打开APK文件
可以看到有其由如下文件组成
- asserts  存放一些配置文件和资源文件，如字体，本地html
- lib so文件，项目自身的so和依赖的三方库中的so，
- classes.dex 代码编译后的字节码，一个dex最多支持65535个方法
- res 资源文件，图片，字符串等等，Raw下一般是音频，xml文件等
- resources.arsc  编译后的二进制资源文件
- META-INF 存放签名信息，用于保证apk包的完整性和系统的安全性
- AndroidManifest.xml

## 优化Asserts
- 删除无用的字体
- 减少iconfont的使用，用svg替代
- 动态下崽资源
- 压缩资源文件
  代码参考Microsoft的codepush项目中的解压zip代码/7z压缩库，hzy3774/AndroidUn7zip


## 优化Lib目录
- 配置ABI Filter
只保留与设备架构相关的so库，可以很大程度的降低lib目录的大小。
```gradle
ndk {
    abiFilters "armeabi"，"armeabi-v7a","x86"
}
```
除了abi配置，还可以配置不同屏幕密度的的打包方式

- 根据cpu 引入so
每一种cpu架构都定义了一种abi，abi定义了其所对应的cpu架构执行二进制so文件的格式规范。
so的优点:让开发者直接利用c 和 c++ 代码；速度快，没有解释编译的开销；内存分配不受单个应用限制，有效减少oom;反编译难度大，安全性较高
缺点：出现崩溃难以定位。
- 动态加载so
so分为动态加载和本地加载，为了减少安装包的大小，可以从云端瞎子后动态加载。本地加载用System.loadLibrary(),动态加载用System.load()
动态加载时需要注意，路径不要是私有路径，即应用包名下的私有目录。

- 避免复制so
系统安装apk的时候不会安装apk里面全部的so，而是根据当前的cpu类型在apk里面复制最合适的so
android:extractNativeLibs = “false”

- 谨慎处理so
abiFilters

## 优化Resource.arsc
改文件是文件映射资源，
- 删除无用的映射
- 进行资源混淆，微信开发团队开源了一个资源混淆工具-AndResGuard,能够将资源的名称进行混淆和缩减
  > 需要注意，当存在用反射获取资源的sdk时需要留意，放入不混淆的白名单

## 优化META-INF
该文件夹下有三个文件，分别是MANIFEST.MF，CERT.SF,CERT.RSA
- MANIFEST.MF 是摘要文件，打包时会遍历所有文件（非文件夹，非签名文件）逐个编码生成摘要信息，并记录于此，如果逆向修改了任何的文件，就会出现文件和摘要信息
  不匹配的情况，导致安全校验失败

- CEFT.SF 该文件是MANIFEST.MF的签名文件，防止对MANIFEST.MF的二次修改
- CERT.RSA 包含了公钥和加密算法等信息，重要信息是对CERT.SF用私钥加密后的值

## 优化Res目录
res目录是瘦身的重头戏
- 通过IDE删除无用资源
在任何资源上右键点击 -> refactor -> remove unused resources,不要勾选@id

- 打包时剔除无用资源
```gradle
buildTypes {
    release {
        zipAlignEnabled true
        minifyEnabled true
        shrinkResources true //去除无效额资源文件，开启混淆后才能生效
    }
}
```

- 删除无用的语言
大部分app其实不需要支持几十种语言，微信也做了根据地区选择下载语言包的功能
作为国内应用，可以只支持中文
```gradle
//...
defaultConfig {
    resConfigs "zh"
}
```

- 控制raw中的资源大小
Raw和Assets可以用来存放资源，二者区别如下
  - Assets目录允许下面有多级子目录，而Raw不允许存在子目录
  - Assets中的文件不会产生R文件映射，所以不能被lint分析，Raw可以
  
存放音频文件时，尽量不要使用无损格式

- 减少Shape文件
统一App的按钮分格
统一颜色，推荐用颜色名命名，不用功能命名颜色
为减少shape，使用tianzhijiexian/SelectorInjection库
google 的 materialButton也可以

- 减少layout文件
复用和融合
使用Shatter

- 动态下载图片

- 分目录放置图片
图片放在drawable下，相当于放在drawable-mdpi中。
微信的放置策略
 - 大量的图片在hdpi和xdpi中
 - 表情图在hdpi中
 - anim中都是xml文件
 - drawable目录中有大量的xml和少量的.9图
 - layout下全是xml
 - raw中放置音频
 - svg可能在raw,assets，drawable中
 - app中最大的文件夹是图片文件夹
 - layout文件夹较小

- 合理使用图片资源
 - 背景图等大图放在xhdpi，xxhdpi
 - 如果某些图片在真机上确实会展示异常，就用多套图适配

- 丢弃特定的资源

- 移除Lib库中的配置文件
  去掉三方库中的一些配置文件，可以在打包时去掉，配置packageOptions{}

## 优化图片资源
- 纯色资源用icon
- 两种以上颜色的icon，用webp
- webp无法达到效果，用png
- 如果没有alpha通道，用jpg
- 1k左右不做优化
如果UI是sketch做的，可以用Android-RES-Export插件，免去二次替换的工作

## 优化dex
