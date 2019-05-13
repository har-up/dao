# Android属性动画

## 介绍

> property  animation(android属性动画)是android API 11加入的新的动画系统。通过属性动画你可以通过指定时间，指定view对象的属性变化来达到任何动画效果。

你可以为属性动画指定下面几个特性：

- Duration:动画时长

- Repeat count and behavior：动画重复次数，以及是否倒放动画

- Animator sets：动画集，同时进行多个属性动画

- Frame refresh delay：延时执行，动画在给定时间后开始执行

  

## 与视图动画的区别

- 视图动画没有属性动画强大，只能进行一些旋转、缩放、位置动画，不提供背景颜色变化动画。

- 视图动画只是视图上面的变化，并没有改变view的真实属性。比如改变位置后，真实的view依然在原处，点击事件还在原处触发。

- 属性动画能实现所有视图动画效果，功能完善健壮，可自定义自由度高。视图动画实现起来代码量少，更方便。如果视图动画能完成你的所有需求就没有必要选择代码量较多的属性动画。

  

## API

文档：https://developer.android.google.cn/reference/android/animation/package-summary

主要的类：

> Animator实现类

- [ValueAnimator](https://developer.android.google.cn/reference/android/animation/ValueAnimator.html)
- [ObjectAnimator](https://developer.android.google.cn/reference/android/animation/ObjectAnimator.html)
- [AnimatorSet](https://developer.android.google.cn/reference/android/animation/AnimatorSet.html)

> Evaluators 估值器

- [IntEvaluator](https://developer.android.google.cn/reference/android/animation/IntEvaluator.html)
- [FloatEvaluator](https://developer.android.google.cn/reference/android/animation/FloatEvaluator.html)
- [ArgbEvaluator](https://developer.android.google.cn/reference/android/animation/ArgbEvaluator.html)
- [TypeEvaluator](https://developer.android.google.cn/reference/android/animation/TypeEvaluator.html)

> Interpolators 插值器

- [AccelerateDecelerateInterpolator](https://developer.android.google.cn/reference/android/view/animation/AccelerateDecelerateInterpolator.html)
- [AccelerateInterpolator](https://developer.android.google.cn/reference/android/view/animation/AccelerateInterpolator.html)
- [AnticipateInterpolator](https://developer.android.google.cn/reference/android/view/animation/AnticipateInterpolator.html)
- [AnticipateOvershootInterpolator](https://developer.android.google.cn/reference/android/view/animation/AnticipateOvershootInterpolator.html)
- [BounceInterpolator](https://developer.android.google.cn/reference/android/view/animation/BounceInterpolator.html)
- [CycleInterpolator](https://developer.android.google.cn/reference/android/view/animation/CycleInterpolator.html)
- [DecelerateInterpolator](https://developer.android.google.cn/reference/android/view/animation/DecelerateInterpolator.html)
- [LinearInterpolator](https://developer.android.google.cn/reference/android/view/animation/LinearInterpolator.html)
- [OvershootInterpolator](https://developer.android.google.cn/reference/android/view/animation/OvershootInterpolator.html)
- [TimeInterpolator](https://developer.android.google.cn/reference/android/animation/TimeInterpolator.html)

### ValueAnimator的用法

- ValueAnimator 分时计值引擎，通过给定的属性值。然后它内部在duration时长内不断地把计算得到的属性值回调到onAnimationUpdate，所以需要设置回调，在回调里给动画的对象设置计算得到的属性值来实现动画效果。(可以通过ofObject(MyTypeEvaluator) MyTypeEvaluator来自定义计算规则。

```kotlin
 var animator = ValueAnimator.ofFloat(0f, 30f, 100f,200f,400f) //给定五个属性值
        animator.addUpdateListener {               //设置回调
            									   //还有一些其他的监听可以设置(在		                                                        Animator类下)...
            view.x = it.animatedValue as Float     //在回调里改变view的属性实现动画效果
        }
        animator.apply {
            duration = 1000          			   //设置动画时长
            repeatCount = ValueAnimator.INFINITE   //设置动画重复次数
            repeatMode = ValueAnimator.REVERSE     //设置动画倒放
            start()                                //开始动画
        }
```

- 通过中间类PropertyValuesHolder

  ```kotlin
  ValueAnimator.ofPropertyValuesHolder(PropertyValuesHolder.ofFloat("x",0f,30f,100f,200f,400f))
  ```

### ObjectAnimator的用法

> 该类是ValueAnimator的子类，可以更方便的使用属性动画

```kotlin
        var animator = ObjectAnimator.ofFloat(view, "x", 0f,100f,110f, 400f)
        animator.apply {
            duration = 1000
            repeatCount = ValueAnimator.INFINITE
            repeatMode = ValueAnimator.REVERSE
            start()
        }
```

> 注意：让ObjectAnimator可以行之有效，view，对象需要有对应属性set方法。比如上面的例子的view必须有setX（）方法,某些属性你可能还需要在set方法里调用invalidate方法。另外在需要的情况下还要有get方法...

### AnimatorSet的用法

```kotlin
val bouncer = AnimatorSet().apply {
    play(bounceAnim).before(squashAnim1)    //动画squashAnim1之前执行bounceAnim动画
    play(squashAnim1).with(squashAnim2)		//两个动画同时执行
    play(squashAnim1).with(stretchAnim1)	
    play(squashAnim1).with(stretchAnim2)
    play(bounceBackAnim).after(stretchAnim2)//在stretchAnim2之后执行bounceBackAnim动画
}
val fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f).apply {
    duration = 250
}
AnimatorSet().apply {                       
    play(bouncer).before(fadeAnim)			// 在fadeAnim之后执行bouncer set动画
    start()
}
```



### LayoutTransition的用法

> 该类可以为ViewGroup添加显示或隐藏动画

方法1、在xml的ViewGroup中设置

```xml
android:animateLayoutChanges="true"
```

方法2：

```kotlin
relativeLayout.layoutTransition = LayoutTransition()
```



### 通过xml文件实现动画

#### [StateListAnimator](https://developer.android.google.cn/reference/android/animation/StateListAnimator.html)

- 新建animator对象xml文件

  在res目录下新建一个目录res/animator,新建一个xml文件res/animator/test_animator.xml

- 根据不同的状态制定不同动画，test_animator.xml文件的root tag写为selecter。如下：

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <selector xmlns:android="http://schemas.android.com/apk/res/android">
      <item android:state_pressed="true">
          <set>
              <objectAnimator
                      android:propertyName="scaleX"   //属性名
                      android:duration="@android:integer/config_longAnimTime"  //时长
                      android:valueTo="1.5"			//属性endValue
                      android:valueType="floatType"/> //属性值类型
              <objectAnimator
                      android:propertyName="scaleY"
                      android:duration="@android:integer/config_longAnimTime"
                      android:valueTo="1.5"
                      android:valueType="floatType"/>
          </set>
      </item>
      <item android:state_pressed="false">
          <set>
              <objectAnimator
                      android:propertyName="scaleX"
                      android:duration="@android:integer/config_longAnimTime"
                      android:valueType="floatType"
                      android:valueTo="1"/>
              <objectAnimator
                      android:propertyName="scaleY"
                      android:valueTo="1"
                      android:duration="@android:integer/config_longAnimTime"
                      android:valueType="floatType"/>
          </set>
      </item>
  </selector>
  ```

- view绑定xml动画对象

  - 在xml文件中绑定

    ```xml
    android:stateListAnimator="@animator/test_animator"
    ```

  - 在代码中加载

    ```kotlin
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                var loadStateListAnimator = AnimatorInflater.loadStateListAnimator(this, R.animator.animate_scale)
                view.stateListAnimator = loadStateListAnimator
            }
    ```

    

#### [AnimatedStateListDrawable](https://developer.android.google.cn/reference/android/graphics/drawable/AnimatedStateListDrawable.html)

- 在drawable/目录下新建xml文件

- 根标签<animated-selector>，状态帧<item>,动画定义<transition>

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <animated-selector xmlns:android="http://schemas.android.com/apk/res/android">
      <item android:id="@+id/state_pressed" android:state_pressed="true" android:drawable="@color/red"/>
  
      <item android:id="@+id/state_focused" android:state_focused="true" android:drawable="@color/black"/>
  
      <item android:id="@+id/defalut" android:drawable="@color/blue" />
  
      <transition
              android:fromId="@id/state_pressed"
              android:toId="@id/state_focused">
          <animation-list>
              <item android:duration="10000" android:drawable="@color/colorPrimary"/>
              <item android:duration="10000" android:drawable="@color/colorPrimaryDark"/>
          </animation-list>
      </transition>
  </animated-selector>
  
  ```

  

