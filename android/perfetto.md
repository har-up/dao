## Perfetto的使用

ferfetto是一个网页版的性能分析工具 [官网](https://links.jianshu.com/go?to=https%3A%2F%2Fui.perfetto.dev%2F%23!%2F)

分析流程如下：

## 耗时分析

### 在代码中增加trace代码

```kotlin
Trace.beginSection("init splash event")
Trace.endSection()
```



### 开启日志抓取

* 手机开发者模式 -> 系统跟踪 -> 开启录制跟踪记录 -> 打开app操作  -> 结束录制  -> 分享日志 -> 网页打开日志



### 命令抓取

* 在官网选择好配置后复制命令新建一个文件.pbtx

* 将文件push到手机

  ```shell
  adb -s e960bdde push C:\Users\Dao\Desktop\default.pbtx /data/misc/perfetto-configs/default.pbtx
  ```

* 通过配置文件执行perfetto命令

  ```shell
  adb -s e960bdde shell perfetto --txt -c /data/misc/perfetto-configs/default.pbtx -o /data/misc/perfetto-traces/trace_file.perfetto-trace
  ```

* 拉取文件

  ```shell
  adb -s e960bdde pull /data/misc/perfetto-traces/trace.file.perfetto-trace
  ```

* 网站打开文件分析