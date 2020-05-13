# Deque
双向队列，线性的集合接口支持从头部和尾部插入和删除元素。继承Queue接口（支持集合两边进行操作的接口,增加/插入/抽取/检查）。

主要接口方法：

- addFirst   在队列头部插入元素作为新的头
- addLast   在队列尾部插入元素作为新的尾部
- offerFirst  在容量允许情况下在头部插入元素。对比addFirst，它有返回值（boolean). 
- offerLast  在容量允许情况下在尾部插入元素。对比addList，它有返回值（boolean).
- removeFirst  检索并删除队列的第一个元素 ，与pollFirst的唯一区别是，在队列为空时会抛出异常
- removeLast  检索并删除队列的尾部元素 ，与pollLast的唯一区别是，在队列为空时会抛出异常  
- pollFirst  检索并删除队列的第一个元素 ，队列为空时返回NULL
- pollLast 检索并删除队列的尾部元素，队列为空时返回NULL
- getFirst  检索获取第一个元素，不删除元素。队列为空抛出异常
- getLast 检索获取尾部元素，不删除元素。队列为空时抛出异常
- peekFirst 检索获取第一个元素，不删除元素，队列为空时返回NULL
- peekLast 检索获取尾部元素，不删除元素，队列为空时返回NULL
- removeFirstOccurence 删除找到的第一个指定元素，找到并删除后返回true
- removeLastOccurence 删除找到的最后一个指定元素, 找到并删除后返回true
- contains 是否存在指定的元素
- size 返回队列元素数量
-  iterator 返回迭代器（从头到尾）
- descendingIterator 返回迭代器（从尾到头）



## ArrayDeque

继承接口Deque，通过数组的形式存放元素，