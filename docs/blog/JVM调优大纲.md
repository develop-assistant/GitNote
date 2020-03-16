# 系列文章

- CPU飙升
- 线程死锁
- OOM
- 内存泄露


# 何时需要调优

- Heap内存（老年代）持续上涨达到设置的最大内存值；
- Full GC 次数频繁；
- GC 停顿时间过长（超过1秒）；
- 应用出现OutOfMemory 等内存异常；
- 应用中有使用本地缓存且占用大量内存空间；
- 系统吞吐量与响应性能不高或下降。

# 调优目标是什么

- 延迟：GC低停顿和GC低频率；
- 低内存占用；
- 高吞吐量;

# 如何分析CPU飙升的问题

# 如何分析内存溢出的问题

- jps
- jstack 19645
- jstack 19645 >t.log 存储进程信息
- jmap -histo 19645 查看内存信息
- jmap -dump:format=b,file=heap.bin 19645 dump内存信息到heap.bin文件
- jstat -gc 19645 jstat 监视垃圾回收（GC）时间，次数
- 分析heap.bin

# 如何分析内存泄露的问题

# 如何分析线程死锁的问题

- top
- jps
- jstack


# 结语