# Divide and Conquer

1. Divide the problem into a number of subproblems that are smaller instances of the same problem.
2. Conquer the subproblems by solving them recursively. If the subproblem sizes are small enough, however, just solve the subproblems in a straightforward manner.
3. Combine the solutions to the subproblems into the solution for the original problem.

## 堆排序

构造完全二叉树

### 自底向上堆调整

定义root节点位置为0，树的大小为k，则从第k/2-1子节点向第0节点进行堆调整。调整堆过程中若发现子树不满足堆条件需要再次递归子节点进行堆调整。

i=k，接着将A[0]根节点与A[i]尾节点进行交换，从i从k到2再次进行堆调整，循环k-1次。

### 算法复杂度

第一次建立堆，时间复杂度为O(n)。每次堆头节点与堆尾部节点进行交换，则堆再次调整复杂度为lg(n)，由于要调整n次，所以时间复杂度为nlg(n)。所以总体上是nlg(n)复杂度。

不需要存额外空间存中间计算结果，所以空间复杂度为O(1)。

### 优先队列

#### 出队

以大顶堆为例，直接取出堆顶节点，就是出队，然后将堆末尾节点赋值到堆顶节点，然后做一次大顶堆计算。因为只做一次顶部的大顶堆计算所以时间复杂度为O(lg(N))

#### 入队

插入节点到到堆尾，然后循环与父节点比较，若比父节点大，则与父节点交换，直到到达root节点

# Dynamic Programing

1. Characterize the structure of an optimal solution.
2. Recursively define the value of an optimal solution.
3. Compute the value of an optimal solution, typically in a bottom-up fashion.
4. Construct an optimal solution from computed information.

# Greedy Algorithms

1. Determine the optimal substructure of the problem.
2. Develop a recursive solution. (For the activity-selection problem, we formulated recurrence (16.2), but we bypassed developing a recursive algorithm based
   on this recurrence.)
3. Show that if we make the greedy choice, then only one subproblem remains.
4. Prove that it is always safe to make the greedy choice. (Steps 3 and 4 can occur
   in either order.)
5. Develop a recursive algorithm that implements the greedy strategy.
6. Convert the recursive algorithm to an iterative algorithm.

# MapReduce

A MapReduce program is composed of a map procedure (or method), which performs filtering and sorting (such as sorting students by first name into queues, one queue for each name), and a reduce method, which performs a summary operation (such as counting the number of students in each queue, yielding name frequencies)

A MapReduce framework (or system) is usually composed of three operations (or steps):

1. Map: each worker node applies the map function to the local data, and writes the output to a temporary storage. A master node ensures that only one copy of redundant input data is processed.
2. Shuffle: worker nodes redistribute data based on the output keys (produced by the map function), such that all data belonging to one key is located on the same worker node.
3. Reduce: worker nodes now process each group of output data, per key, in parallel.

# BitMap

利用bit位的0,1，做一些节省内存空间和提升运算性能。大部分用于内容匹配，查找，去重的运算。

# 布隆过滤器

布隆过滤器（英语：Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用**于检索一个元素是否在一个集合中**。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。

本质上布隆过滤器是一种数据结构，比较巧妙的概率型数据结构（probabilistic data structure），特点是高效地插入和查询，可以用来告诉你 **“某样东西一定不存在或者可能存在”**。

相比于传统的 List、Set、Map 等数据结构，它更高效、占用空间更少，但是缺点是其返回的结果是概率性的，而不是确切的。

https://zhuanlan.zhihu.com/p/43263751

## 原理

布隆过滤器的原理是，当一个元素被加入集合时，通过K个散列函数将这个元素映射成一个位数组中的K个点，把它们置为1。检索时，我们只要看看这些点是不是都是1就（大约）知道集合中有没有它了：如果这些点**有任何一个0**，则被检元素**一定不在**；如果**都是1**，则被检元素**很可能在**。这就是布隆过滤器的基本思想。

### ## 实现

//todo

采用redis的bitmap实现。

[布隆过滤器 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8)

# 回溯法

回溯算法实际上一个类似枚举的搜索尝试过程，主要是在搜索尝试过程中寻找问题的解，当发现已不满足求解条件时，就“回溯”返回，尝试别的路径。

回溯法是一种选优搜索法，按选优条件向前搜索，以达到目标。但当探索到某一步时，发现原先选择并不优或达不到目标，就退回一步重新选择，这种走不通就退回再走的技术为回溯法，而满足回溯条件的某个状态的点称为“回溯点”。

# 树

## B树

在B树中，内部（非叶子）节点可以拥有可变数量的子节点（数量范围预先定义好）。当数据被插入或从一个节点中移除，它的子节点数量发生变化。为了维持在预先设定的数量范围内，内部节点可能会被合并或者分离。因为子节点数量有一定的允许范围，所以B树不需要像其他自平衡查找树那样频繁地重新保持平衡，但是由于节点没有被完全填充，可能浪费了一些空间。子节点数量的上界和下界依特定的实现而设置。例如，在一个2-3 B树（通常简称2-3树），每一个内部节点只能有2或3个子节点。

B树中每一个内部节点会包含一定数量的键，键将节点的子树分开。例如，如果一个内部节点有3个子节点（子树），那么它就必须有两个键： a1 和 a2 。左边子树的所有值都必须小于 a1 ，中间子树的所有值都必须在 a1 和a2 之间，右边子树的所有值都必须大于 a2 。

### B树插入

节点最多有3个孩子 (Knuth 阶为 3).

![image](https://upload.wikimedia.org/wikipedia/commons/3/33/B_tree_insertion_example.png)

## B+树

A B+ tree can be viewed as a B-tree in which each node contains only keys (not key–value pairs), and to which an additional level is added at the bottom with linked leaves.

The primary value of a B+ tree is in storing data for efficient retrieval in a block-oriented storage context — in particular, filesystems. This is primarily because unlike binary search trees, B+ trees have very high fanout (number of pointers to child nodes in a node,[1] typically on the order of 100 or more), which reduces the number of I/O operations required to find an element in the tree.

### Overview

The order, or branching factor, b of a B+ tree measures the capacity of nodes (i.e., the number of children nodes) for internal nodes in the tree. The actual number of children for a node, referred to here as m, is constrained for internal nodes so that [b/2]{ceilling} <= M <= b. The root is an exception: it is allowed to have as few as two children.

### Implementation

**The leaves** (the bottom-most index blocks) of the B+ tree are often **linked to one another in a linked list**; this makes range queries or an (ordered) iteration through the blocks simpler and more efficient (though the aforementioned upper bound can be achieved even without this addition). This does not substantially increase space consumption or maintenance on the tree. This illustrates one of the significant advantages of a B+tree over a B-tree; in a B-tree, since not all keys are present in the leaves, such an ordered linked list cannot be constructed. A B+tree is thus particularly useful as a database system index, where the data typically resides on disk, as it allows the B+tree to actually provide an efficient structure for housing the data itself (this is described in[4]:238 as index structure "Alternative 1").

### 插入

节点要处于违规状态，它必须包含在可接受范围之外数目的元素。

首先，查找要插入其中的节点的位置。接着把值插入这个节点中。
如果没有节点处于违规状态则处理结束。

如果某个节点有过多元素，则把它分裂为两个节点，每个都有最小数目的元素。在树上递归向上继续这个处理直到到达根节点，如果根节点被分裂，则创建一个新根节点。为了使它工作，元素的最小和最大数目典型的必须选择为使最小数不小于最大数的一半。

把键1-7连接到值 d1-d7 的B+树。链表（红色）用于快速顺序遍历叶子节点。树的分叉因子 {\displaystyle b} b=4。

## B*树

B*树是B+树的变种，相对于B+树他们的不同之处如下：

1. 首先是关键字个数限制问题，B+树初始化的关键字初始化个数是cei(m/2)，b*树的初始化个数为（cei(2/3*m)）

2. B+树节点满时就会分裂，而B*树节点满时会检查兄弟节点是否满（因为每个节点都有指向兄弟的指针），如果兄弟节点未满则向兄弟节点转移关键字，如果兄弟节点已满，则从当前节点和兄弟节点各拿出1/3的数据创建一个新的节点出来；

特点：

在B+树的基础上因其初始化的容量变大，使得节点空间使用率更高，而又存有兄弟节点的指针，可以向兄弟节点转移关键字的特性使得B*树额分解次数变得更少；

![image](http://hi.csdn.net/attachment/201106/7/8394323_13074405869mSW.jpg)

## Differences between B+ trees and B trees.

### Advantages of B+ trees:

Because B+ trees don't have data associated with interior nodes, more keys can fit on a page of memory. Therefore, it will require fewer cache misses in order to access data that is on a leaf node.
The leaf nodes of B+ trees are linked, so doing a full scan of all objects in a tree requires just one linear pass through all the leaf nodes. A B tree, on the other hand, would require a traversal of every level in the tree. This full-tree traversal will likely involve more cache misses than the linear traversal of B+ leaves.

### Advantage of B trees:

Because B trees contain data with each key, frequently accessed nodes can lie closer to the root, and therefore can be accessed more quickly.

![image](https://upload.wikimedia.org/wikipedia/commons/thumb/3/37/Bplustree.png/400px-Bplustree.png)

### 参考文献

[MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

[B树-Wikipedia](https://zh.wikipedia.org/wiki/B%E6%A0%91)

[B+Tree-Wikipedia](https://en.wikipedia.org/wiki/B%2B_tree#Overview)

[平衡二叉树、B树、B+树、B*树 理解其中一种你就都明白了](https://zhuanlan.zhihu.com/p/27700617)

[MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

[布隆过滤器-Wikipedia](https://zh.wikipedia.org/wiki/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8)

# Log-structured merge-tree

In computer science, the log-structured merge-tree (or LSM tree) is a data structure with performance characteristics that make it attractive for providing indexed access to files with **high insert volume**, such as transactional log data.

磁盘随机操作缓慢，但当顺序访问时很快，不管磁盘是固态还是磁性材料。这种影响在主存较小。

LSM树不同于有一个大的索引结构来存下批量的写操作（像机关枪一样在磁盘中扫描或者增加大量的写操作）。它顺序的，写入到一系列的更小的所有文件中。所以每个文件包含了一批在**一段短时间内的变化**。每个文件在被写入前都被**排序**以便于稍微快速的检索。这些文件都是不变的；他们从来不更新。每次更新都写入新的文件。**会检查所有文件来定期合并**，以降低文件的数量。

当更新到达时，他们被添加到一个**内存缓冲区，通常是作为一棵树（红黑等等）来保证键值顺序**。这"内存表"在大部分实现里以预写日志（WAL）的方式复制到磁盘中，以用作恢复。当排序的数据充满了内存表，会被刷入硬盘的一个新文件。这样的过程随着越来越多的写入不断重复着。文件无法被编辑,所以系统只做顺序IO。新的条目或者简单的编辑会建立新的文件.

旧文件不会去更新重复的条目来取代先前的记录（或被删除的记录）。所以开始出现了一些冗余。

系统定期的执行压缩操作。压缩操作把很多文件合并起来，消除其中重复的更新或者删除操作。这对于消除上面提到的冗余问题非常重要，但是更重要的是，通过减少文件的增长量来保证读操作的性能。由于文件被排序了，所以合并文件的过程是相当高效的。

**当请求读操作时，系统首先检查缓冲区（内存表）**。如果这些关键字没找到，那么会一个接着一个反序检查一系列磁盘文件，直到关键字被找到。**每个文件都被顺序排列所以很容易遍历**，然而当文件越来越多的时候，会变得越来越慢，这是因为需要检查每一个文件。

所以光看读性能，LSM比他的就地修改的同胞们慢一些。幸运的是有一些技巧可以提高性能。最常见的方法是在内存建立页索引，这提供了一个让你更接近目标关键字的查找。

即使有压缩操作，读操作依然会访问多个文件。大部分实现通过布隆解析器(Bloom filter)来避免这一点。

压缩算法有基本压缩法和分级压缩法。

# SkipList

## 参考文献

[Log Structured Merge Trees译文以及LSM调研心得](http://weakyon.com/2015/04/08/Log-Structured-Merge-Trees.html)

# HyperLogLog

# LRU

LRU 缓存机制可以通过哈希表辅以双向链表实现，我们用一个哈希表和一个双向链表维护所有在缓存中的键值对。

双向链表按照被使用的顺序存储了这些键值对，靠近头部的键值对是最近使用的，而靠近尾部的键值对是最久未使用的。

哈希表即为普通的哈希映射（HashMap），通过缓存数据的键映射到其在双向链表中的位置。

这样以来，我们首先使用哈希表进行定位，找出缓存项在双向链表中的位置，随后将其移动到双向链表的头部，即可在 O(1) 的时间内完成 get 或者 put 操作。具体的方法如下：

对于 get 操作，首先判断 key 是否存在：

如果 key 不存在，则返回 -1−1；

如果 key 存在，则 key 对应的节点是最近被使用的节点。通过哈希表定位到该节点在双向链表中的位置，并将其移动到双向链表的头部，最后返回该节点的值。

对于 put 操作，首先判断 key 是否存在：

如果 key 不存在，使用 key 和 value 创建一个新的节点，在双向链表的头部添加该节点，并将 key 和该节点添加进哈希表中。然后判断双向链表的节点数是否超出容量，如果超出容量，则删除双向链表的尾部节点，并删除哈希表中对应的项；

如果 key 存在，则与 get 操作类似，先通过哈希表定位，再将对应的节点的值更新为 value，并将该节点移到双向链表的头部。

上述各项操作中，访问哈希表的时间复杂度为 O(1)O(1)，在双向链表的头部添加节点、在双向链表的尾部删除节点的复杂度也为 O(1)O(1)。而将一个节点移到双向链表的头部，可以分成「删除该节点」和「在双向链表的头部添加节点」两步操作，都可以在 O(1)O(1) 时间内完成。

一个建议

在双向链表的实现中，使用一个伪头部（dummy head）和伪尾部（dummy tail）标记界限，这样在添加节点和删除节点的时候就不需要检查相邻的节点是否存在。

## 参考

[lruhuan-cun-ji-zhi-by-leetcode-solution](https://leetcode-cn.com/problems/lru-cache/solution/lruhuan-cun-ji-zhi-by-leetcode-solution/)
