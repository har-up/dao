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
