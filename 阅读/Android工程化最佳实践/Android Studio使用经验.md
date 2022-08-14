## 调试
- 查看日志 使用自带的logcat
- 检查布局
使用facebook的sonar库，litho库，layout inspector inplugin

- 网络抓包
Sonar中实时的查看当前app的网络请求，添加插件 NetworkSonarPlugin()
> 仅仅支持OkHttp

## 插件
### 统计相关
- WakaTime
可以统计在不同项目，不同语言上工作的时间

- statistic
代码规模统计插件，详细的给出工程的代码量，方法数等

- MetricsReloaded
代码统计工具

- Resource Usage Count
将资源的引用数实时的显示在左边

- Drawable Preview
可以看到图片资源的预览图

- color manager
颜色管理

- layout master
快照微调

### 工具相关
- jvm debuuger memory view
可以通过该插件看到堆里Object的数目

- DTOnator
Json转换工具

- Android Code Generator
代码模板

- Gradle Killer
gradle有时会卡死导致studio也卡住，可以kill java.exe。
该插件只是调用了命令行，其源代码可以作为插件开发的范例

- Translate plugin
翻译功能

- setting sync
插件同步工具
