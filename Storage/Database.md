# 事务

## 支持事务的数据库特性

### 原子性(Atomicity)

原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚。

### 一致性(Consistency)

一致性是指事务必须使数据库从一个一致的状态变到另外一个一致的状态，也就是执行事务之前和之后的状态都必须处于一致的状态。

### 隔离性(Isolation)

隔离性是指当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离

### 持久性(Durability)

持久性是指一个事务一旦被提交了，那么对于数据库中的数据改变就是永久性的，即便是在数据库系统遭遇到故障的情况下也不会丢失提交事务的操作。

## 事务级别

### 未提交读（Read uncommited）

所有事务都可以看到其他未提交事务的执行结果。会出现脏读现象。

### 已提交读（Read commited）

一个事务只能看见已经提交事务所做的改变。会出现不可重复读现象。

### 可重复读（Repeatable read）

Allows **only committed data to be read** and further requires that, **between two reads of a data item by a transaction, no other transaction is allowed to update it**. However, the transaction may not be serializable with respect to other transactions. For instance, when it is searching for data satisfying some conditions, a transaction may find some of the data inserted by a committed transaction, but may not find other data inserted by the same
transaction. 会出现幻读现象。

### 可串行化（Serializable）

最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。

**Mysql的默认隔离级别就是Repeatable read。**

## 事务隔离级别的实现

详见MySqlPrinciple.md的MVCC介绍。

### Locking

Instead of locking the entire database, a transaction could, instead, lock only those
data items that it accesses. 

Further improvements to locking result if we have two kinds of locks: shared
and exclusive. **Shared locks** are used for data that the transaction reads and
**exclusive locks** are used for those it writes. Many transactions can hold shared
locks on the same data item at the same time, but a transaction is allowed an
exclusive lock on a data item only if no other transaction holds any lock (regardless
of whether shared or exclusive) on the data item.

#### Lock-Based Protocols

1. Shared. If a transaction Ti has obtained a shared-mode lock (denoted by S)
   on item Q, then Ti can read, but cannot write, Q.
2. Exclusive. If a transaction Ti has obtained an exclusive-mode lock (denoted
   by X) on item Q, then Ti can both read and write Q.

### Timestamps

Timestamps are used to ensure that transactions access each data item in order of the transactions’ timestamps if their accesses conflict. When this is not possible, offending transactions are aborted and restarted with a new timestamp.

#### The Timestamp-Ordering Protocol

The timestamp-ordering protocol ensures that any conflicting read and write
operations are executed in timestamp order. This protocol operates as follows:

1. Suppose that transaction Ti issues read(Q).
- a. If **TS(Ti) < W-timestamp(Q)**, then Ti needs to read a value of Q that
  was already overwritten. Hence, the read operation is **rejected**, and Ti
  is rolled back.
- b. If **TS(Ti) ≥ W-timestamp(Q)**, then the read operation **is executed**, and
  R-timestamp(Q) is set to the maximum of R-timestamp(Q) and TS(Ti).
2. Suppose that transaction Ti issues write(Q).
- a. If **TS(Ti) < R-timestamp(Q)**, then the value of Q that Ti is producing
  was needed previously, and the system assumed that that value would
  never be produced. Hence, the system **rejects** the write operation and
  rolls Ti back.
- b. If **TS(Ti) < W-timestamp(Q),** then Ti is attempting to write an obsolete
  value of Q. Hence, the system **rejects** this write operation and rolls Ti
  back.
- c. **Otherwise**, the system **executes the write** operation and sets W-timestamp(Q) to TS(Ti).

### Multiple Versions and Snapshot Isolation

Timestamps are used to ensure that transactions access each data item in order of the transactions' timestamps if their accesses conflict. When this is not possible, offending transactions are aborted and restarted with a new timestamp.

In snapshot isolation, we can imagine that each **transaction is given its own version, or snapshot**, of the database when it begins. It reads data from this private version and is thus isolated from the updates made by other transactions. If the transaction updates the database, that update appears only in its own version, not in the actual database itself. Information about these updates is saved so that the **updates can be applied to the "real" database** if the **transaction commits**.

## 没有事务隔离的问题

### 脏读（Dirty Read）

一个事务处理过程里读取了另一个未提交的事务中的数据。

### 不可重复读（NonRepeatable Read）

对于数据库中的某个数据，一个事务范围内多次查询却返回了不同的数据值，这是由于在查询的间隔期间，另外一个事务修改并提交了该数据。锁定正在读取的行。

### 幻读（Phantom Read）

是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行。所以第一个事务出现幻读现象，因为没有发现第二个事务更新的数据。锁定所读取的所有行。

A *phantom read* occurs when, in the course of a transaction, new rows are added or removed by another transaction to the records being read.

This can occur when range locks are not acquired on performing a SELECT ... WHERE* operation. **The *phantom reads* anomaly is a special case of *Non-repeatable reads* when Transaction 1 repeats a ranged *SELECT ... WHERE* query and, between both operations, Transaction 2 creates (i.e. INSERT) new rows (in the target table) which fulfill that *WHERE* clause.**

| Transaction 1                                                             | Transaction 2                                                                    |
| ------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `/* Query 1 */ SELECT * FROM users WHERE age BETWEEN 10 AND 30; `         |                                                                                  |
|                                                                           | `/* Query 2 */ INSERT INTO users(id, name, age) VALUES (3, 'Bob', 27); COMMIT; ` |
| `/* Query 1 */ SELECT * FROM users WHERE age BETWEEN 10 AND 30; COMMIT; ` |                                                                                  |

Note that Transaction 1 executed the same query twice. If the highest level of isolation were maintained, the same set of rows should be returned both times, and indeed that is what is mandated to occur in a database operating at the SQL SERIALIZABLE isolation level. However, at the lesser isolation levels, a different set of rows may be returned the second time.

In the SERIALIZABLE isolation mode, Query 1 would result in all records with age in the range 10 to 30 being locked, thus Query 2 would block until the first transaction was committed. In REPEATABLE READ mode, the range would not be locked, allowing the record to be inserted. Therefore, the second statement of Query 1 would not return the same result as the first one.

[Phantom_reads_Isolation (database systems)](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Phantom_reads)

### 不可重复读和幻读到底有什么区别呢？

(1) 不可重复读是读取了其他事务更改的数据，**针对update操作**

解决：使用**行级锁**，锁定该行，事务A多次读取操作完成后才释放该锁，这个时候才允许其他事务更改刚才的数据。

(2) 幻读是一个事务两次读取范围查询，第二次读取了其他事务新增的数据，也就是第一次读和第二次读的结果不一样，**针对insert和delete操作**

解决：使用表级锁，锁定整张表，事务A多次读取数据总量之后才释放该锁，这个时候才允许其他事务新增数据。

这时候再理解事务隔离级别就简单多了呢。

[快速理解脏读、不可重复读、幻读和MVCC](https://cloud.tencent.com/developer/article/1450773)

## 两段锁协议(Two-phase locking)

By the 2PL protocol, locks are applied and removed in two phases:

1. **Expanding phase** (aka Growing phase): locks are acquired and no locks are released (the number of locks can only increase).
2. **Shrinking phase** (aka Contracting phase): locks are released and no locks are acquired.

Two major types of locks are used:

- **Write-lock** (**exclusive lock**) is associated with a database object by a transaction (Terminology: "the transaction locks the object," or "acquires lock for it") before ***writing* (inserting/modifying/deleting)** this object.
- **Read-lock** (**shared lock**) is associated with a database object by a transaction before ***reading*** (retrieving the state of) this object.

The common interactions between these lock types are defined by blocking behavior as follows:

- An existing ***write-lock*** on a database object **blocks an intended *write*** upon the same object (already requested/issued) by another transaction by blocking a respective *write-lock* from being acquired by the other transaction. The second write-lock will be acquired and the requested write of the object will take place (materialize) after the existing write-lock is released.
- A ***write-lock* blocks an intended (already requested/issued) *read*** by another transaction by blocking the respective *read-lock* .
- A ***read-lock* blocks an intended *write*** by another transaction by blocking the respective *write-lock*.
- A ***read-lock* does not block an intended *read*** by another transaction. The respective *read-lock* for the intended read is acquired (shared with the previous read) immediately after the intended read is requested, and then the intended read itself takes place.

| Lock type  | read-lock | write-lock |
| ---------- | --------- | ---------- |
| read-lock  | **✔**     | **X**      |
| write-lock | **X**     | **X**      |

[Two-phase locking - Wikipedia](https://en.wikipedia.org/wiki/Two-phase_locking)

## 死锁

### MySQL 8死锁范例

#### 共享锁和排它锁死锁场景

| 时间序列 | 事务1                                                        | 事务2                                                                                                              |
| ---- | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
|      | begin;                                                     | begin;                                                                                                           |
| 1    | select * from user where id = 2 lock in share mode;(获取共享锁) |                                                                                                                  |
| 2    |                                                            | select * from user where id = 2 lock in share mode;(获取共享锁)                                                       |
| 3    | update user set age = age+1 where id = 2;                  |                                                                                                                  |
| 4    | 获取排他锁失败，于是等待。                                              |                                                                                                                  |
| 5    |                                                            | update user set age = age+1 where id = 2;                                                                        |
| 6    |                                                            | ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction。获取排他锁失败，MySQL检测到死锁，放弃该事务。 |
| 7    | 因为事务2检测到死锁而放弃锁，故此时事务1获取了排它锁，唤醒并继续走一步流程。                    |                                                                                                                  |

#### 排它锁死锁场景

| 时间序列 | 事务1                                                       | 事务2                                                                                                                       |
| ---- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
|      | begin;                                                    | begin;                                                                                                                    |
| 1    | update user set age = age + 1 where id = 1;(获取id=1记录的排他锁) |                                                                                                                           |
| 2    |                                                           | update user update age = age + 1 where id = 2;(获取id=2记录的排他锁)                                                              |
| 3    | update user set age = age + 1 where id = 2;               |                                                                                                                           |
| 4    | 获取id=2记录的排他锁失败，于是等待。                                      |                                                                                                                           |
| 5    |                                                           | update user set age = age + 1 where id = 1;                                                                               |
| 6    |                                                           | ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction。获取id = 1记录的排他锁失败，MySQL检测到死锁，放弃该事务。 |
| 7    | 因为事务2检测到死锁而放弃锁，故此时事务1获取了id = 2记录的排它锁，唤醒并继续走一步流程。          |                                                                                                                           |

### 死锁解决方法

### 如何尽可能避免死锁

1. 合理的设计索引，区分度高的列放到组合索引前面，使业务 SQL 尽可能通过索引`定位更少的行，减少锁竞争`。

2. 调整业务逻辑 SQL 执行顺序， 避免 update/delete 长时间持有锁的 SQL 在事务前面。

3. 避免`大事务`，尽量将大事务拆成多个小事务来处理，小事务发生锁冲突的几率也更小。

4. 以`固定的顺序`访问表和行。比如两个更新数据的事务，事务 A 更新数据的顺序为 1，2;事务 B 更新数据的顺序为 2，1。这样更可能会造成死锁。

5. 在并发比较高的系统中，不要显式加锁，特别是是在事务里显式加锁。如 select … for update 语句，如果是在事务里`（运行了 start transaction 或设置了autocommit 等于0）`,那么就会锁定所查找到的记录。

6. 尽量按`主键/索引`去查找记录，范围查找增加了锁冲突的可能性，也不要利用数据库做一些额外额度计算工作。比如有的程序会用到 “select … where … order by rand();”这样的语句，由于类似这样的语句用不到索引，因此将导致整个表的数据都被锁住。

7. 优化 SQL 和表设计，减少同时占用太多资源的情况。比如说，`减少连接的表`，将复杂 SQL `分解`为多个简单的 SQL。

##### 参考：

[阿里二面：怎么解决MySQL死锁问题的？](https://xie.infoq.cn/article/41285fabb8c4ca612d150b415)

## 为什么推荐InnoDB存储引擎使用自增主键

InnoDB使用聚集索引，数据记录本身被存于主索引（一颗B+Tree）的叶子节点上。这就要求同一个叶子节点内（大小为一个内存页或磁盘页）的各条数据记录按主键顺序存放，因此每当有一条新的记录插入时，MySQL会根据其主键将其插入适当的节点和位置，如果页面达到装载因子（InnoDB默认为15/16），则开辟一个新的页（节点）。

这样就会**形成一个紧凑的索引结构**，近似顺序填满。由于每次**插入时也不需要移动已有数据**，因此效率很高，也不会增加很多开销在维护索引上。

如果使用非自增主键（如果身份证号或学号等），由于每次插入主键的值**近似于随机**，因此每次新纪录都要被**插到现有索引页得中间**某个位置.

此时MySQL不得不为了**将新记录插到合适位置而移动数据**，甚至目标页面可能已经被回写到磁盘上而从缓存中清掉，此时又要从磁盘上读回来，这增加了很多开销，同时**频繁的移动、分页**操作造成了**大量的碎片**，得到了**不够紧凑的索引结构**，后续不得不通过OPTIMIZE TABLE来重建表并优化填充页面。

因此，只要可以，请尽量在InnoDB上采用自增字段做主键。

[为什么推荐InnoDB引擎使用自增主键？](http://www.ywnds.com/?p=8735)

## 聚集索引和非聚集索引

**非聚集索引**索引项顺序存储，但索引项对应的内容却是随机存储的。

**聚集索引**：索引键值的逻辑顺序与索引所服务的表中相应行的物理顺序相同的索引，被称为**聚集索引**，反之为**非聚集索引**。聚集索引因为与表的元组物理顺序一一对应，所以只有一种排序，即一个数据表只有一个聚集索引。一般是表中的主键索引，如果表中没有显示指定主键，则会选择表中的第一个不允许为NULL的唯一索引，如果还是没有的话，就采用Innodb存储引擎为每行数据内置的6字节ROWID作为聚集索引。

---

https://zhuanlan.zhihu.com/p/50597960)

---

# MongoDB

## Description

MongoDB is a cross-platform, document oriented database that provides, high performance, high availability, and easy scalability. MongoDB works on concept of collection and document.

跨平台，面向文档性数据库。高性能，高可用，易扩展。基于集合与文档概念工作。不支持数据库事务操作。

Database is a physical container for collections

Collection is a group of MongoDB documents

A document is a set of key-value pairs

## MongoDB部署方案

MongoDB的集群部署方案中有三类角色：实际数据存储结点、配置文件存储结点和路由接入结点。

连接的客户端直接与路由结点相连，从配置结点上查询数据，根据查询结果到实际的存储结点上查询和存储数据。MongoDB的部署方案有单机部署、复本集（主备）部署、分片部署、复本集与分片混合部署。

## 路由接入结点

## 配置文件存储结点

存储配置文件的服务器其实存储的是片键与chunk以及chunk与server的映射关系。

## 实际数据存储结点

### 数据存储特点

类似JSON数据结构。

### 写：

数据写如主节点，然后异步复制到从节点。

### 读

从主或从节点读取。

---

# Redis

## Introduction

Redis is a very fast non-relational database that stores a mapping of keys to five different types of values. Redis supports **in-memory persistent storage on disk**, **replication to scale read performance**, and **client-side sharding to scale write performance**. 

## Other features

### Persistence

Redis has two different forms of persistence available for writing in-memory data to disk in a compact format. 

1. The first method is a point-in-time dump either when certain conditions are met (a number of writes in a given period) or when one of the two dump-to-disk commands is called.

2. The other method uses an append-only file that writes every command that alters data in Redis to disk as it happens. Depending on how careful you want to be with your data, append-only writing can be configured to never sync, sync once per second, or sync at the completion of every operation.

### Read and write

To support higher rates of read performance (along with handling
failover if the server that Redis is running on crashes), Redis supports master/slave replication where slaves connect to the master and receive an initial copy of the full database. As writes are performed on the master, they’re sent to all connected slaves for updating the slave datasets in real time. With continuously updated data on the slaves, clients can then connect to any slave for reads instead of making requests to the master. 

---

# Index architecture/Indexing Methods

## Non-clustered

The data is present in arbitrary order, but the logical ordering is specified by the index. The data rows may be spread throughout the table regardless of the value of the indexed column or expression. The non-clustered index tree contains the index keys in sorted order, with the leaf level of the index containing the pointer to the record (page and the row number in the data page in page-organized engines; row offset in file-organized engines).

In a non-clustered index,

The physical order of the rows is not the same as the index order.
The indexed columns are typically non-primary key columns used in JOIN, WHERE, and ORDER BY clauses.
There can be more than one non-clustered index on a database table.

## Clustered

Clustering alters the data block into a certain distinct order to match the index, resulting in the row data being stored in order. Therefore, **only one clustered index can be created on a given database table**. Clustered indices can greatly increase overall speed of retrieval, but usually only where the data is accessed sequentially in the same or reverse order of the clustered index, or when a range of items is selected.

Since the physical records are in this sort order on disk, the next row item in the sequence is immediately before or after the last one, and so fewer data block reads are required. The primary feature of a clustered index is therefore the ordering of the physical data rows in accordance with the index blocks that point to them. Some databases separate the data and index blocks into separate files, others put two completely different data blocks within the same physical file(s).

## Cluster

When multiple databases and multiple tables are joined, it's referred to as a cluster (not to be confused with clustered index described above). The records for the tables sharing the value of a cluster key shall be stored together in the same or nearby data blocks. This may improve the joins of these tables on the cluster key, since the matching records are stored together and less I/O is required to locate them.[2] The cluster configuration defines the data layout in the tables that are parts of the cluster. A cluster can be keyed with a B-Tree index or a hash table. The data block where the table record is stored is defined by the value of the cluster key.

# Column order

The order that the index definition defines the columns in is important. It is possible to retrieve a set of row identifiers using only the first indexed column. However, it is not possible or efficient (on most databases) to retrieve the set of row identifiers using only the second or greater indexed column.

## 联合索引

首先介绍一下联合索引。联合索引其实很简单，相对于一般索引只有一个字段，联合索引可以为多个字段创建一个索引。它的原理也很简单，比如，我们在（a,b,c）字段上创建一个联合索引，则索引记录会首先按照A字段排序，然后再按照B字段排序然后再是C字段，因此，联合索引的特点就是：

第一个字段一定是有序的，当第一个字段值相等的时候，第二个字段又是有序的，比如下表中当A=2时所有B的值是有序排列的，依次类推，当同一个B值得所有C字段是有序排列的。

## 索引合并

1. 索引合并是把几个索引的范围扫描合并成一个索引。
2. 索引合并的时候，会对索引进行并集，交集或者先交集再并集操作，以便合并成一个索引。
3. 这些需要合并的索引只能是一个表的。不能对多表进行索引合并。

## 索引覆盖

## 区分度

选择性（区分度）是指不重复的列值个数/列值的总个数，一般意义上建索引的字段要区分度高，而且在建联合索引的时候区分度高的列字段要放在前边，这样可以在第一个条件就过滤掉大量的数据。

---

# 数据库算法

## 外排序

外排序（External sorting）是指能够处理极大量数据的排序算法。通常来说，外排序处理的数据不能一次装入内存，只能放在读写较慢的外存储器（通常是硬盘）上。外排序通常采用的是一种“排序-归并”的策略。在排序阶段，先读入能放在内存中的数据量，将其排序输出到一个临时文件，依此进行，将待排序数据组织为多个有序的临时文件。尔后在归并阶段将这些临时文件组合为一个大的有序文件，也即排序结果。

//todo 算法解释

## Merge Join（归并连接）

也成为Sorted merge join算法。
Two-Phase Multiway Merge-Sort算法的具体描述分为2个阶段，如下所示

Phase 1

- Fill main memory with records.
- Sort with favorite main memory sorting algorithms.
- Write sorted list to disk.
- Repeat until all records have been put into one of the sorted lists.

Phase 2

- Initially load input buffers with the first block of their respective sorted lists.
- Repeated run a competition among the first unchosen records of each of the buffered blocks.
- If an input block is exhausted, get the next block from the same file.
- If the output block is full, write it to disk.

## Hash Join

Hash joins are typically more efficient for larger result sets than nested loops, and can only be used for **equi joins.**

Hash joins require an equijoin predicate (a predicate comparing values from one table with values from the other table **using the equals operator '='**).

### Classic hash join

The classic hash join algorithm for an inner join of two relations proceeds as follows:

First prepare a hash table of the smaller relation. The hash table entries consist of the join attribute and its row. Because the hash table is accessed by applying a hash function to the join attribute, it will be much quicker to find a given join attribute's rows by using this table than by scanning the original relation.

Once the hash table is built, scan the larger relation and find the relevant rows from the smaller relation by looking in the hash table.
The first phase is usually called the "build" phase, while the second is called the "probe" phase. Similarly, the join relation on which the hash table is built is called the "build" input, whereas the other input is called the "probe" input.
This algorithm is simple, but it requires that the smaller join relation fits into memory, which is sometimes not the case. A simple approach to handling this situation proceeds as follows:

<pre>
1. For each tuple r in the build input R
  1. Add r to the in-memory hash table
  2. If the size of the hash table equals the maximum in-memory size:
    1. Scan the probe input S, and add matching join tuples to the output relation
    2. Reset the hash table, and continue scanning the build input R
2. Do a final scan of the probe input S and add the resulting join tuples to the output relation
</pre>

### Grace hash join

Refer to Wikipedia.

### Hybrid hash join

Refer to Wikipedia.

---

# 三大范式

## 第一范式

强调的是列的原子性，即列不能够再分成其他几列。

考虑这样一个表：【联系人】（姓名，性别，电话） 
如果在实际场景中，一个联系人有家庭电话和公司电话，那么这种表结构设计就没有达到 1NF。要符合 1NF 我们只需把列（电话）拆分，即：【联系人】（姓名，性别，家庭电话，公司电话）

## 第二范式

它的规则是要求资料表里的所有资料都要和该资料表的键（主键与候选键）有完全依赖关系：每个非键属性必须独立于任意一个候选键的任意一部分属性。如果有哪些资料只和一个键的一部分有关的话，就得把它们独立出来变成另一个资料表。如果一个资料表的键只有单个字段的话，它就一定符合第二范式。

一个数据表匹配第二范式当且仅当

- 它匹配第一正规化

- 所有非键字段都不能是候选键非全体字段的函数
  
  <pre>
  组件ID(主键) | 价格     | 供应商ID(主键)  |    供应商名称       | 供应商住址
  73             | 20.00   |  1             |  Stylized Parts    |  VA
  65             | 69.99   |  2             |  ACME Industries   |  CA
  65             | 59.99   |  1             |  Stylized Parts    |  VA
  </pre>
  
  如上面例子的供应商名称和供应商地址与供应商ID相关，也就是和主键其中的子集相关，所以就不符合第二范式，需要拆除供应商信息。

检查数据表里的每个字段，确认它们是不是都和关键词完全相关， 这样才能知道这个数据表是不是匹配第二范式； 如果不是的话，就把那些不完全相关的字段移到独立的数据表里。 

## 第三范式

要求所有非键属性都只和候选键有相关性，也就是说非键属性之间应该是独立无关的。

# 关系键

## 主键

主键，又称主码（英语：primary key或unique key）。数据库表中对储存数据对象予以唯一和完整标识的数据列或属性的组合。一个数据表只能有一个主键，且主键的取值不能缺失，即不能为空值（Null）。

## 超键

超键（英语：superkey），有的文献称“超码”，是在数据库关系模式设计中能够唯一标示多元组（即“行”）的属性集。

包含所有属性的集叫做明显（平凡）超键。

## 候选键

在关系模型中，候选键或候选码（英语：candidate key）是某个关系变量的一组属性所组成的集合，它需要同时满足下列两个条件：

- 这个属性集合始终能够确保在关系中能唯一标识元组
- 在这个属性集合中找不出真子集能够满足条件(1)

满足第一个条件的属性集合称为超键，因此我们也可以把候选键定义为“最小超键”，也就是不含有多余属性的超键。

## 外键

外键（英语：foreign key，台湾译外来键，又称外部键）。其实在关系数据库中，每个数据表都是由关系来连系彼此的关系，父数据表（Parent Entity）的主键（primary key）会放在另一个数据表，当做属性以创建彼此的关系，而这个属性就是外键。

## 代理建

在关系型数据库设计中，代理键（英语：surrogate key）是在当数据表中的候选键都不适合当主键时，例如数据太长，或是意义层面太多，就会请一个无意义的但唯一的字段来代为作主键。

# 参考文献

[mysql 里创建‘联合索引’的意义](https://www.jianshu.com/p/XgXfhf)

[数据结构与算法（7）：数据库索引原理及优化](https://plushunter.github.io/2017/07/21/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%EF%BC%887%EF%BC%89%EF%BC%9A%E6%95%B0%E6%8D%AE%E5%BA%93%E7%B4%A2%E5%BC%95%E5%8E%9F%E7%90%86%E5%8F%8A%E4%BC%98%E5%8C%96/)

[数据库内部排序算法之两阶段多路归并排序算法实现](https://cloud.tencent.com/developer/article/1135261)

[外排序-wikipedia](https://zh.wikipedia.org/wiki/%E5%A4%96%E6%8E%92%E5%BA%8F)

[Hash Join-Wikipedia](https://en.wikipedia.org/wiki/Hash_join)

[据库范式概念解析（第一范式，第二范式，第三范式）](https://caoyanbao.iteye.com/blog/562290)

[数据库规范化](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%A7%84%E8%8C%83%E5%8C%96)

[关系键](https://zh.wikipedia.org/wiki/%E5%85%B3%E7%B3%BB%E9%94%AE)