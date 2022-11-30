## adb 
adb 是一款对安卓应用进行渗透测试的必备工具。Android SDK默认自带adb，它位于Android SDK的platform-tools目录中。
在安装SDK的过程中，我们已经将其路径添加到了系统环境变量中。

### 检查已连接的设备
```shell
adb devices
```

### 启动shell
```shell
adb shell
```
该命令将为已连接的设备打开一个shell

当真机和模拟器同时连接时
```shell
//打开模拟器shell
adb -e shell

//打开真机shell
adb -d shell

//打开指定设备
adb -s [设备名称]
```

### 列出软件包
当使用adb连接到android设备的shell时，可以使用shell中的工具与设备进行交互。
```shell
pm list packages
```

### 推送文件到设备
我们可以将计算机中的文件推到设备上
```shell
adb push [本地文件路径] [设备上路径]

adb push test.txt /data/local/tmp
```

### 从设备中拉取文件
```shell
adb pull [设备上的文件]
```

### 安装应用
```shell
adb install 文件名.apk
```

### adb连接排除故障
adb经常无法识别模拟器，即使模拟器运行正常。
可以运行
```shell
adb kill-server
```
结束设备上的adb daemon，并将它重启。



