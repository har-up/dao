# LruCache源码分析

## android.util.LruCache

- **构造**

  给定一个缓存的最大值，构造一个LruCache实例。实例构造时初始化一个LinkedHashMap对象map和maxSize;

- **put**

  - key和value都不能为空，否则抛出空指针异常；

  - 内部维护的条目的的size+1。通过map.put把条目放进map中，如果有旧值，则会新值替换吊旧值并且返回旧值previous，否则返回null。再对返回的值进行判断，若为null则size-1。

  - 如果存放的条目数量size大于maxSize时。会把最后一个值给删掉，evictionCount+1这个步骤的效率不高（得到entrysets遍历到最后一个值获取key，然后通过map.remove(key)来删除最后一个放入最久的那个条目）。
