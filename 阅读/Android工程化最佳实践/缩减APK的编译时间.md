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

## 编译环境优化
- 升级硬件设备
建议用m.2接口的ssd 和 16 * 2的3600内存，cpu尽量选用可以超频的
- 升级软件
新版本的gradle要比旧版本的快

- 优化工程配置
开启offlie后，所有的编译操作都会走B本地缓存，可以极大的减少编译时间

- 配置studio的可用内存
默认情况下stuio的最大堆内存为1280m,help->edit custom vm options -> studio.vmoptions文件，该配置加大内存 -xmx

- 提升jvm的堆内存
默认情况下，最低堆内存为1536m,grale目前支持dexing-in-progress和增量编译，dexing-in-progress要求最堆大小最小为2048m，所以推荐将其设置为2gb
> org.gradle.jvmargs=-xmx2048m -xx.maxpersize:512 ....

- 开启并行编译
gradle.properties中 org.gradle.parallel = true

- 启用demand模式
gradle.properties中配置 org.gradle.configureondemand = true

- 配置dexoptions
在DexOptions中可以指定编译时并行的进程数
```gradle
android {
    dexOptions {
        preDexLibraries true //预编译依赖，可实现增量build,clean操作会很慢
        maxProcessCount 8  //最大进程数，默认值4
        javaMaxHeapSize "2048" //编译时使用的最大堆内存
    }
}
```

- 善用缓存
  - 减少动态方法
    比如动态生成版本号
  - 硬编码BuildConfig 和 Res
  - 拆分脚本
  - 拆分代码
  - 写死库的版本号

## 精简工程
- 差异化加载plugin
常用的build相关插件有:tiny,buildtime,dexcount
优化使用：每次发版前开启tiny压缩一次图片；在需要检测和优化build的时候启用buildtime，其他时候关闭；release包中才开启dexcount

- 使用webp 和 svg

- 精简语言和图片资源
通过resConfig做资源过处理

- 善用no-op

- exclude无用库

- 删减Module

- 去掉Multiex

- 删除无用资源

## 综合技巧
- 构建开发的flavor
- 优化multiDex
- 跳过无用的task
  lint的tash时间占80%,如果是debug时不在意lint结果，可以跳过lint的task。
  > gradle build -x lint -x lintVitalRelease //跳过lint和lintVitalRelease的task
- 关闭aapt的图片优化
  aapt是android资源打包工具，每次打包aapt会自动完成图片的压缩，在开发过程中不在意包的大小可以关闭该功能
  ```gradlle
  android {
      if(project.hasProperty('devBuild'){
        splits.abi.enable = false
        splits.density.enable = false
        aaptOptions { cruncherEnable false}
    }  
  }
  ```

- 谨慎使用aspectJ
AspectJ是实现Aop的工具之一，可以无侵入的插入一些代码，常用作日志埋点，性能监控，动态权限控制等
正因为AspectJ会在build时进行代码的插入，所以build时间会增加

## 多渠道打包工具
- MultiChannelPackageTool  在zip文件的comment区域插入标识，适用于v1 签名。打包速度快。不支持v2.0签名

- 美团的walle 
  v2.0签名将apk包区分四大块，contents of zip entries;signing Block;central directory; end of central directory;
  签名方案是将标识加在signing block中，只有这个区域的二次修改可以绕过签名校验机制。
  使用方式：
  ```gradle
  walle {
      apkOutputFolder = new File("${project.buildDir}/outputs/channels")
      apkFileNameFormat = '${appName}-${channel}-v${versionName}.apk'
      channelFile = new File("${project.getProjectDir()}/channel")
      
  }
  ```
  在java代码中通过下面代码获取channel信息
  ```java
  String channel = WalleChannelReader.getChannel(this.getApplicationContext());
  ```
  打包命令：assemble${variantName}Channels 进行打包
  如：gradlew clean assembleDevReleaseChannels

- 腾讯的vasDolly
和美团的方案差不多，支持v1和v2签名