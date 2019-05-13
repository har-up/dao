# 自定义View流程

### 一、继承LinearLayout

​	继承LinearLayout并生成四个构造函数，少参数构造函数逐级调用多一个参数的构造函数，在最多参数的构造函数中执行初始化操作。

### 二、初始化操作

​	获取自定义View属性，有下面两种类型

 -  自定义属性

   - 在values目录下新建attrs.xml文件

   - 定义自定义view的属性

     ```xml
     <?xml version="1.0" encoding="utf-8"?>
     <resources>
         <declare-styleable name="DropMenuView">— 自定义view属性集名称
             <attr name="bg_color" format="color"></attr>-自定义属性名称 format-该属性类型 
         </declare-styleable>
     </resources
     ```

   - 在构造器中通过传过来的参数获取对应的属性

     1、获取TypeArray对象 

     ```kotlin
     var a = context.obtainStyledAttributes(attrs, R.styleable.DropMenuView)
     ```

     2、通过TypeArray对象获取属性

     ```kotlin
     mBackground = a.getColor(R.styleable.DropMenuView_bg_color, Color.WHITE)
     ```

 - android自带属性

   获取自带属性和自定自定义类似，只不过key值为android前缀



### 三、自定义View代码编写

 - View的绘制与填充

   - 采用填充的方式来完成自定义View的显示

     一般通过LayoutInfalter.from(context).inflate(...)的方式来填充

   - 通过重写onDraw()方法完成View的绘制

   > 如果不能正常的显示绘制或填充的视图，有必要重写onMeasure或onLayout对绘制或填充内容进行测量和布局编排

 - 触摸反馈

   编写接口完成一些点击事件的回调

### 遇到的问题

#### 1、填充layout文件到自定义view时，填充view显示异常

> ```kotlin
> val view:View = LayoutInflater.from(mContext).inflate（layoutResId,null）
> this.add(view)
> ```
>
> 使用该类方式填充view到自定义viewgroup中，显示出来的东西很难看

**解决办法**：更换填充方式

> ```kotlin
> val view:View = LayoutInflater.from(mContext).inflate(R.layout.item_menu,this,false)
> this.add(view)
> ```



