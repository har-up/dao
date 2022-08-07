## 使用场景
- 日夜间模式
Fragment的设计是为了数据和视图分离
  Fragment的生命周期剖析
  - 数据的初始化onCreate在视图初始化onCreateView之前
  - 数据的销毁onDestroy在视图销毁onDestroyView之后
  - 通过fm.detach和fm.attach的commit操作可以直接刷新视图而不影响数据
  > 根据这些特点，我们可以实现在不重启activity的日夜间模式切换，这可以作为种方案

- 缓存界面数据
无论在ViewPager还是Activity中，Fragment都可以缓存内部的数据，可以用来在动态的场景中将稳定的数据和不稳定的视图进行分离
> Fragment view的初始化可以放在onCreateView或者onActivityCreated中进行，两者都是官方给出的正规写法

- 作为Presenter
Activity的生命周期改变会影响到Fragment的生命周期，但Fragment的生命周期改变只和当前执行的操作有关
所以我们可以用Fragment来监听Activity的状态，很多时候可以通过一个没有UI的Fragment让内部逻辑和Activity解耦
Google在Android源码中也使用了这一小技巧：
reportFragment类,这个类的作用就是用于分发生命周期
> Glide中也是用的这种无UI的Fragment来监听状态的，当监听到Activity销毁是自动结束图片的下载操作

## 源码分析
- Transaction
Fragment的控制由一个事务控制，事务的概念是计算机应用中不可或缺的组件模型，它拨正了用户操作的原子性，一致性，隔离性和永久性。事物的提交是一次性的，有回滚机制，
可以在一次执行多个操作后统一提交。不过Fragment管理的FragmetnTransaction没有任何和事物回滚相关的代码，更多的是和栈相关的代码，所以不是一个标准事务的实现。
  
- 提交操作
FragmentTransaction的异步提交是通过Handler来实现的，会等待UI线程空闲后再进行提交。
如果Fragment想要入栈，不能使用commitNow();
> 如果Fragment需要在短时间内高频率的执行add和remove操作，那么建议用同步操作，即commitNow
  - commitAllowtateLoss
  commit（）是严格保留状态的，在日常开发中经常会遇到如下错误
  ```java
  can not perorm this action after onSaveInstanceState
  ```  
  意思是Fragment被展示或取消时，如果Activity已经被用户finish，那么就会出现这个线上崩溃。
  为了避免这个错误，需要在执行操作前阙波Activity没有被退出或销毁，但commit是异步的，所以很难判断这一点，所以在不得已额情况下，可以使用commitAllowtateLoss替代commit
  > google在后续的版本中添加了isStateSaved方法用于判Activity是否执行过onSaveInstanceState,所以可以在commit前可以通过这个判读

- Add操作的原理
Fragment的添加本质上就是添加view，但其中的生命周期管理很复杂。
在Activity的onResume执行后我们进行Fragment的添加操作，Activity的生命周期不会变，而Fragment则会不断的追逐Activity的生命周期，直到赶上Activity。
如果在Activity的onCreate中添加Fragment，则Fragment的生命周期状态State会跟随Activity的周期逐级加1,

- Replace的本质
replace是两个Fragment之间的操作,其内部就是进行了add和remove两个过程
A-onAttach -> A-onCreate -> B-onPause -> B-onStop -> B-onDestroyView -> B-onDestroy -> B-onDetach -> A-onViewCreated -> A-onActivityCreated -> A- 
onStart -> onResume

- Fragment的可见性监听
Fragment的hide和show只是调用了View的setVisibility（）,并不会调用Fragment的生命周期。我们可以通过fragment的isVisible和isHidden来判断Fragment的展示状态。
> 新版本中也直接可以在onResume和onPause中监听展示，隐藏。

- ViewPager中的懒加载
Viewpager和Fragment结合使用可以通过setUserVisibleHint（）回调监听Fragment的可见性。
> 新版本中也直接可以在onResume和onPause中监听展示，隐藏。

## 常见问题
- Activity为空
当Fragment已经执行了onDetach，调用getActivity就会返回null，可以每次都进行一个判空操作避免错误；也可以在Fragment的onDestroy中取消异步监听；
在Fragment中还提供了一个getContext（）,所以可以优先使用getContext(),必要情况下才使用getActivity

- startActivityForResult
不要使用getActivity.startActivity，直接使用Fragment的startActivity,否则会监听不到回调的结果

- ViewPager的getItem
不要使用ViewPager的adapter的getItem来获取当前展示的Fragment，因为该方法可能会返回一个新的Fragment，并不是当前的Fragment。

- DialogFragment
DialogFragment的显示操作也是一个事务，如果点击一次按钮就调用show,在连续，快速点击时，会导致同一时刻多次触发show操作，进而引发崩溃；可以对按钮做点击事件的排重处理规避；也可以将DialogFragment作为变量，判断其状态。isAdd()  isShowing（）

- 重叠显示的问题
切换横竖屏时，每次切换都会执行一次Fragment创建和add操作，导致Fragment重叠。
可以通过saveInstanceState进行判断，如果Fragmetn为null才加载

## Fragment的替代品
- Square的Flow
- 自定义View替代
- Shatter库


