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
