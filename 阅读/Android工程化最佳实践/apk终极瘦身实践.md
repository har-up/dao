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
