# Drawable下的图片文件内存占用
最近面试有几家都问起了Drawable下的文件加载的内存计算，很明显我没能回答上来，所以在这里彻底弄懂这个问题并记录下来。

## 实践
为了加深印象，当然是得手动实践一下。首先准备一张图片（我选的是一张1080*1440 126k)，逐次放在drawable-hdpi,drawable-xhdpi,drawable-xxhdpi
下进行实验。

- 获取加载后的drawable大小的方式
  ```kotlin
   private fun printBitmapSize(drawable: Drawable) {
        drawable as BitmapDrawable
        if (drawable == null) return
        Log.d(this.javaClass.name,"count ${drawable.bitmap.byteCount}")
        Log.d(this.javaClass.name,"width ${drawable.bitmap.width}")
        Log.d(this.javaClass.name,"height ${drawable.bitmap.height}")
    }
  ```
- 获取设备显示相关信息
  ```kotlin
     var displayMetrics = DisplayMetrics()
     windowManager.defaultDisplay.getMetrics(displayMetrics);
     displayMetrics.apply {
            Log.d(this@BitmapActivity.javaClass.name,"densityDpi ${this.densityDpi}")
            Log.d(this@BitmapActivity.javaClass.name,"heightPixels ${this.heightPixels}")
            Log.d(this@BitmapActivity.javaClass.name,"widthPixels ${this.widthPixels}"
            Log.d(this@BitmapActivity.javaClass.name,"scaledDensity ${this.scaledDensity}")
     }
  ```
## 实验结果
通过上面的只更换图片文件放在不同的目录下得到了一下的实验结果

实验图片： 1080*1440 126k

设备：dpi:440 | height: 2068 | width: 1080

| bitmap | hdpi（240dpi） | xhdpi（320dpi) | xxhdpi(480dpi) |
|:-----------|:------------|:-----------|:-----------|
| byte count | 20908800 | 11761200 | 5227200 |
| width | 1980 | 1485 | 990 |
| height | 2640 | 1980 | 1320 |

- 我的设备的dpi是440dpi，信息意味着系统会首先去drawable-xxhdpi目录下找资源文件（最接近的原则）。
- 通过比较计算可以得到 
   width之间的关系
  |:---------------------|
  |990 = 440 / 480 * 1080|
  |1495 = 440 / 320 * 1080|
  |1980 = 440 / 240 * 1080|
  
  所以可以得出结论：
  - 加载的drawable下内存 = 设备的dpi/对应drawable的dpi * （图片width * 图片height * 4）   
    |drawable目录|对应dpi|
    |:------------|:----------|   
    |drawable-ldpi | 120 |
    |drawable-mdpi | 160 |
    |drawable-hdpi | 240 |
    |drawable-xhdpi | 320 |
    |drawable-xxhdpi | 480 |
    |drawable-xxxhdpi |640 |
  - 加载的内存大小与图片大小无太大关系，因为图片文件是压缩后的数据，只与图片的分辨率相关。如果完全不适配的话，那么内存占用就是
    图片的width * 图片的height * 4（ARGB8888)
    
## 总结
google采用多目录的目的是 不同分辨率的设备可以设计出同样的效果，比如一个图标在任何设备中都显示为屏幕大小的十分之一。那么就需要使用某种策略
（以dpi为准）对图片进行缩放扩大，当匹配到的文件目录dpi是比当前设备小的那么就要按照比例扩大，如果匹配到的文件目录dpi是比当前设备大的就要
按照比例缩小。

 
