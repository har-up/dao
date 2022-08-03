## 基本概念
- Log的等级
  - Verbose 当前运行时可有可无的信息
  - Debug 调试阶段的数据，一般用于开发定位问题
  - Info 常规信息，一般将比较重要但数据量不大的信息定义为该级别
  - Warn 警告信息，警示作用
  - Error 严重错误信息，需要及时修复

- 命令行操作Log
adb logcatde 命令格式
```shell
logcat [option] ... [filter-spec] ...
```
option:
 - -s:设置输出日志的tag
 - -v:设置日志的输入格式
 - -c:清空缓存日志
 - -d:输出缓存日志后就结束
 - -t:输出最近的几行日志
 - -g:查看日志缓冲区的信息
 - -d:加载一个日志缓冲区
 - -B:以二进制形式输出日志的内容

过滤日志
 - 过滤级别
   logcat *:级别
 - 过滤tag
   logcat ddd:E *:S
   多个：logcat ddd:E aaa:D *:S
 - 过滤关键字
   logcat | grep ddd
   logcat | findstr ddd

 - sublime的过滤插件FilterLies；Highlighter

## Android中的Log
 - 设置代码模板方便编写log打印代码
 - 热部署Log
   利用debug的异常断点和条件断点可以实现debug时的无入侵log (右键单击断点，高级编辑，取消suspend)

## 微信的Xlog
xlog是微信团队开源的组件库Mars中一个模块
- 用C++写核心功能，减少java的损耗
- 对日志进行加密
- 如果想要在Release中打Log，引入xlog是不二之选

## 美团日志库 Logan

## 扩展Log的功能
良好的Log封装库需具备功能
 - 自动生成Tag，默认把当前类名作为tag
 - 允许手动改写tag和添加tag的前缀
 - 能打印LIST,MAP,JSON,POJO对象
 - 要和系统log有区别，输入直观，方便定位
 - Log信息过长后会自动换行，避免丢失信息
 - 打印当前线程名，方便定位
 - Log后面加Log位置，方便点击跳转
 - Release包中不能泄漏debug用到的log
 - Release版本中可以通过混淆从根本上清空log
 - 能自动try-catch的crash上传到云端，能进行线上分析
 - 能在运行时动态修改Log的输出级别，做动态开关，便于和远程用户进行调试

- Tag的自动化
 - 使用类名作为tag
 - 在base类中给Tag赋值
 - 通过堆栈信息获取当前类名（有性能消耗，发布版本不要使用该方法）

- 文本内容设计
 - 提升字符拼接效率，使用string.format()方法
 - 支持超长的信息 

- 开关的设计
 - 自动化/强制开关
   不能仅仅通过true/false来设定开关，还要考虑到原生Log等级；而且只通过true/false还很容易被逆向改造

 - 通过proguard剔除log
   自定义log打印类，然后通过proguard将该类相关的代码彻底清除

 - 不要做过度优化
   不要在release中把所有日志都去掉

## Timber，LogDelegate,Logger
Timber是一个log封装类，还提供了Log分发的能力
LogDelegate目前学习试用，真实项目中推荐用Logger
Logger目前除了不支持自动tag外，几乎支持前边列出的偶有扩展功能

## 日志上报服务Crashlytics

## 实用日志
 - 操作耗时日志
   Android系统有现成的监控耗时的类——timingLogger

 - 页面跳转日志
  在Activity创建的监听器中增加log

 - 网络请求日志
  - jgilfelt/Chuck：yige okhtto的拦截器，可以将请求和响应信息展示在手机上
  - OkLog只打印Url,点击url跳转静态服务器