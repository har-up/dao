# IO
传统的文件输入输出
FileInputStream()
FileOutputStream()
InputStreamReader()
OutputStreamWriter()

# NIO
支持双向，支持非阻塞式读写
使用Buffer缓冲
可以操作Buffer

# OKIO
基于共享的缓冲区设计（以Segment作为存储结构,Segment在Segment线程池中以单链表存在以便复用，在Buffer中以双向链表存在存储数据，head指向头部，是最老的数据）
超时机制