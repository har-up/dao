## 分析项目现状
- gradle profile
在执行编译任务的时候，加上--profile 可以得到此次build的统计报表。
在终端执行如下命令
```shell
gradlew clean
gradlew assembleFlavorDebug --profile //clean后 强制进行全量build
```
执行完后，可以通过/buildsrc/build/reports/profile/profile-time.html文件查看报表

- BuildTimTracker
BuildTimTracker 是可以检查build耗时的gradle插件，会实时的显示每个task的耗时，最终生成十分美观的图表

- Dexcount GradlePlugin
一个可以监控Dex大小和方法数的gradle插件