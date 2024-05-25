## ADB命令

## 查看内存

```shell
adb shell cat /proc/meminfo
adb shell cat proc/meminfo

//列出前10内存消耗进程
adb shell -m 10 
```

## 查看设备ID(IMEI)地址

```shell
adb shell
su
service call iphonesubinfo 1
```

## 屏幕密度

```shell
adb shell wm density
//修改
adb shell wm density 440
//恢复
adb shell wm density reset
```

## 屏幕分辨率

```
adb shell wm size
//修改
adb shell wm size 1920x1080
//恢复
adb shell wm size reset
```

## CPU

```shell
adb shell cat /proc/cpuinfo
```

## 绘制耗时统计

```shell
adb shell dumpsys gfxinfo <包名>
```

## 日志抓取

- 过滤字段抓取

  ```shell
  adb logcat | findstr <filter>
  adb logcat | grep <filter>
  adb logcat -s <filter>
  ```

- TAG过滤

  ```shell
  adb logcat TAG:D *:S
  ```

  D —— Debug

  I —— Info

  W —— Warning

  E —— Error

  F —— Fatal  致命

  S —— Silent（最高，啥也不输出）

- logcat 脚本

  ```shell
  @echo.
  @echo on	
  @echo.	------------开始抓取日志------------------
  @echo.	Ctrl+C停止抓取
  adb logcat -c
  adb logcat -v time >.\LOG"%date:~0,4%-%date:~5,2%-%date:~8,2% %time:~0,2%时%time:~3,2%分%time:~6,2%.txt"
  pause
  ```

- 设置系统日志捕获级别

  ```shell
  adb shell 
  setprop persist.log.tag V/D/I/W/E
  ```




## 启动组件

```shell
adb shell am start -n com.shuwei.sscm/com.shuwei.sscm.ui.brand.BrandVidPlayActivity -e key_ref_id 1
```

## 无线调试

```shell
adb tcpip 555
adb connect 局域网ip
```



## 查看进程优先级

```shell
adb shell
##获取进程ID
ps -A | grep com.pst.orange
##查看进程优先级
cat /proc/【pid】/oom_adj 
```

## 点击事件

```shell
adb shell input tap x y
```

## 滑动事件

```she
abd shell input swipe fromx fromy tox toy
adb shell input swipe 0 0 300 300
```

## 模拟点击键盘按钮

```shell
adb shell input keyevent [key值]
```

| keyevent | 效果                     |
| -------- | ------------------------ |
| 3        | Home键                   |
| 4        | 返回键                   |
| 5        | 拨号键                   |
| 6        | 挂机键                   |
| 19       | 向上                     |
| 20       | 向下                     |
| 21       | 向左                     |
| 22       | 向右                     |
| 24       | 音量加                   |
| 25       | 音量减                   |
| 26       | 电源                     |
| 27       | 拍照（需要在相机应用里） |
| 64       | 打开浏览器               |
| 66       | 回车键                   |
| 67       | 退格键                   |
| 82       | 菜单键                   |
| 84       | 搜索键                   |
| 85       | 播放/暂停键              |
| 86       | 停止播放                 |
| 87       | 播放上一首               |
| 88       | 播放下一首               |
| 92       | 向上翻页                 |
| 93       | 向下翻页                 |
| 112      | 删除键                   |
| 115      | 大写锁定键               |
| 122      | 光标移动到开始键         |
| 123      | 光标移动到末尾键         |
| 164      | 静音                     |
| 168      | 放大键                   |
| 169      | 缩小键                   |
| 176      | 打开系统设置             |
| 187      | 切换应用                 |
| 220      | 降低屏幕亮度             |
| 221      | 提高屏幕亮度             |
| 223      | 系统休眠                 |
| 224      | 点亮屏幕                 |
| 231      | 打开语音助手             |

## 向屏幕輸入信息

```shell
db shell input text [字符串信息]
```



## 清空应用数据

```shell
adb shell pm clear packagename
```

## 清空授权拒绝标志

```shell
adb shell pm clear-permission-flags PACKAGE_NAME PERMISSION_NAME user-set user-fixed
 ```

## Android 12及以上触摸事件屏蔽
关闭：
```shell
# A specific app
adb shell am compat disable BLOCK_UNTRUSTED_TOUCHES com.example.app

# All apps
# If you'd still like to see a Logcat message warning when a touch would be
# blocked, use 1 instead of 0.
adb shell settings put global block_untrusted_touches 0
```

恢复默认 屏蔽
```shell
# A specific app
adb shell am compat reset BLOCK_UNTRUSTED_TOUCHES com.example.app

# All apps
adb shell settings put global block_untrusted_touches 2
```