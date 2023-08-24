## 流程
### Glide的使用
Glide.with(context).load(url).into(target);
- Glide.with(contex)
  1.内部有判断Glide单例是否存在的逻辑，如果glide实例为null，则会走创建新实例的逻辑。其中有个细节是会通过反射获取是否有自定义的AppGlideModule，如果有则反射生成一个自定义的AppGlideModule实例；
  则会走一下自定义appGlideModule的registerComponents回调。

  2.会返回一个RequestManager()。如果context是Activity,Fragment,那么通过对应的fm去获取RequestManager（通过fm添加一个空白的fragment,然后通过这个空白的fragment入参到RequestManagerFactory.build（）来生成与context绑定的RequestManager。
### 4级缓存
- 活动资源 (Active Resources) - 现在是否有另一个 View 正在展示这张图片？
  在当前活动的activity或fragment中正在展示view，则会将其放在activeResource中，其中有一个死循环的线程不断地扫描去通过判断移除缓存。HashMap
- 内存缓存 (Memory cache) - 该图片是否最近被加载过并仍存在于内存中？ LruCache
- 资源类型（Resource） - 该图片是否之前曾被解码、转换并写入过磁盘缓存？EncodeJob()
- 数据来源 (Data) - 构建这个图片的资源是否之前曾被写入过文件缓存？DecodeJob()
有比较多的executor