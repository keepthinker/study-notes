# LevelDB

LevelDB是一个可持久化的KV数据库引擎。

做存储的同学都很清楚，对于普通机械磁盘顺序写的性能要比随机写大很多。比如对于15000转的SAS盘，4K写IO， 顺序写在200MB/s左右，而随机写性能可能只有1MB/s左右。而LevelDB的设计思想正是利用了磁盘的这个特性。 LevelDB的数据是存储在磁盘上的，采用LSM-Tree的结构实现。LSM-Tree将磁盘的随机写转化为顺序写，从而大大提高了写速度。为了做到这一点LSM-Tree的思路是将索引树结构拆成一大一小两颗树，较小的一个常驻内存，较大的一个持久化到磁盘，他们**共同维护一个有序的key空间**。写入操作会首先操作内存中的树，随着内存中树的不断变大，**会触发与磁盘中树的归并操作**，而归并操作本身仅有顺序写。

## 主要特性

下面是LevelDB官方对其特性的描述，主要包括如下几点： 1. key和value都是任意长度的字节数组； 2. entry（即一条K-V记录）默认是按照key的字典顺序存储的，当然开发者也可以重载这个排序函数； 3. 提供的基本操作接口：Put()、Delete()、Get()、Batch()； 4. 支持批量操作以原子操作进行； 5. 可以创建数据全景的snapshot(快照)，并允许在快照中查找数据； 6. 可以通过前向（或后向）迭代器遍历数据（迭代器会隐含的创建一个snapshot）； 7. 自动使用Snappy压缩数据； 8. 可移植性；

[rocksdb](http://alexstocks.github.io/html/rocksdb.html)

[ LevelDB深入浅出之整体架构](https://zhuanlan.zhihu.com/p/67833030)

[LevelDB 和 RocksDB 结构详解 | Daemon.D.Blog](https://daemondshu.github.io/2019/03/21/Programming/Data%20Structure/LevelDB_RocksDB/)
