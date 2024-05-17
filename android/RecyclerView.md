# RecyclerView
## 緩存
RecyclerView有四级缓存
### mAttachedScrap和mChangedScrap
缓存当前屏幕可见的ViewHolder,数据结构是ArrayList()
* mAttachedScrap
缓存被remove的item ViewHolder，数据被更新的item ViewHolder 或 没有更新的ViewHolder以及没有动画或动画可被重用的ViewHolder
* mChangedScrap
依然attach RecyclerView不满足上面条件的item ViewHolder会被放入该列表。

## ReyclerView更新流程
在滑动手势时，会触发onToucheEvent，Move事件触发scrollByInternal（）-> layoutManager.scrollBy() -> fill() -> offset()
RecyclerView的缓存和复用就在fill()逻辑中