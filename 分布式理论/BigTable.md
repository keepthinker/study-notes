# BigTable
## Data Model
A Bigtable is a sparse, distributed, persistent multidimensional sorted map. The map is indexed by a **row key, column key, and a timestamp**; each value in the map is an uninterpreted array of bytes.

(row:string, column:string, time:int64) -> string

### Rows
The row keys in a table are arbitrary strings (currently up to 64KB in size, although 10-100 bytes is a typical size for most of our users). **Every read or write** of data under
a single row key **is atomic** (regardless of the number of different columns being read or written in the row).

Bigtable maintains data **in lexicographic order by row key**. **The row range** for a table is dynamically **partitioned**. Each row range is called a tablet, which is **the unit of distribution and load balancing**.

### Column Families
**Column keys** are **grouped into sets** called column families, which form the basic unit of access control. **All data** stored in a column family is usually **of the same type** (we compress data in the same column family together). A column family must be created before data can be stored under any column key in that family; after a family has
been created, any column key within the family can be used. It is our intent that the number of distinct column families in a table be small (in the hundreds at most), and that families rarely change during operation. In contrast, a table may have an **unbounded number of columns**.

A column key is named using the following syntax: family:qualifier. Column family names must be printable, but qualifiers may be arbitrary strings.

Access control and both disk and memory accounting are performed at the column-family level.

### Timestamps
Each cell in a Bigtable can contain **multiple versions of the same data**; these versions are indexed by timestamp.

To make the management of versioned data less onerous, we support two per-column-family settings that tell Bigtable to **garbage-collect cell versions automatically**. The client can specify either that only the last n versions
of a cell be kept, or that only new-enough versions be kept (e.g., only keep values that were written in the last seven days)

## Building Blocks
Bigtable is built on several other pieces of Google infrastructure. Bigtable uses the distributed **Google File System** (GFS) to store log and data files. 

The Google SSTable file format is used internally to store Bigtable data. An SSTable provides a **persistent**, **ordered immutable map** from keys to values, where both keys and values are arbitrary byte strings. Internally, **each SSTable contains a sequence of blocks** (typically each block is 64KB in size, but this is configurable). A block index (stored at the end of the SSTable) is used to locate blocks; the index is loaded into memory when the SSTable is opened.

Bigtable **relies on a highly-available and persistent distributed lock service** called **Chubby**. A Chubby service consists of five active replicas, **one of which is elected to be the master** and actively serve requests. 

Bigtable uses Chubby for a variety of tasks: to ensure that there is at most one active master at any time; to store the bootstrap location of Bigtable data; to discover tablet servers and finalize tablet server deaths (see Section 5.2); to store Bigtable schema information (the column family information for each table); and to store access control lists.

## Implementation
The Bigtable implementation has three major compo- nents: **a library that is linked into every client**, one **master server**, and many **tablet servers**.

The master is responsible for **assigning tablets to tablet servers**, **detecting the addition and expiration of tablet servers**, **balancing tablet-server load**, and **garbage collection of files in GFS**.

Each tablet server manages a set of tablets. The tablet server **handles read and write requests to the tablets** that it has loaded, and also **splits tablets that have grown too large**.

As with many single-master distributed storage systems, client data does not move through the master: **clients communicate directly with tablet servers for reads and writes**.

A **Bigtable cluster** stores **a number of tables**. **Each table** consists of **a set of tablets**, and **each tablet** contains **all data associated with a row range**.

## Tablet Location
We use a three-level hierarchy analogous to that of a B+- tree to store tablet location information.

![image](D:\git\study-notes\分布式理论\big-table-tablet.jpg)

The first level is a file stored in Chubby that contains the location of the root tablet. The root tablet contains the location of all tablets in a special METADATA table. Each METADATA tablet contains the location of a set of user tablets. The root tablet is just the first tablet in the METADATA table.

The client library caches tablet locations.

## Tablet Assignment
Each tablet is assigned to one tablet server at a time.

Bigtable uses Chubby to keep track of tablet servers. When a tablet server starts, it creates, and acquires an exclusive lock on, a uniquely-named file in a specific Chubby directory. **The master monitors this directory (the servers directory) to discover tablet servers.** A tablet server stops serving its tablets if it loses its exclusive lock: e.g., due to a network partition that caused the server to lose its Chubby session. (Chubby provides an efficient mechanism that allows a tablet server to check whether it still holds its lock without incurring network traffic.) A tablet server will attempt to reacquire an ex- clusive lock on its file as long as the file still exists. A tablet server will attempt to reacquire an exclusive lock on its file as long as the file still exists. If the file no longer exists, then the tablet server will never be able to serve again, so it kills itself.

Once a server’s file has been deleted, the master can move all the tablets that were previously assigned to that server into the set of unassigned tablets.

## Tablet Serving
The persistent state of a tablet is stored in GFS.
Updates are committed to a commit log that stores redo records. Of these updates, the recently committed ones are stored in memory in a sorted buffer called a memtable; the older updates are stored in a sequence of SSTables.

## Compactions
When the memtable size reaches a threshold, the memtable is frozen, a new memtable is created, and the frozen memtable is converted to an SSTable and written to GFS. This minor compaction process has two goals: **it shrinks the memory usage of the tablet server**, and **it reduces the amount of data that has to be read from the commit log** during recovery if this server dies.

## Refinements
### Locality groups
A separate SSTable is generated for each locality group in each tablet. Segregating column families that are not typically accessed together into sep arate locality groups enables more efficient reads.

### Compression
Clients can control whether or not the SSTables for a locality group are compressed.

### Caching for read performance
To improve read performance, tablet servers use two lev- els of caching. **The Scan Cache** is a higher-level cache that caches the key-value pairs returned by the SSTable interface to the tablet server code. **The Block Cache** is a lower-level cache that caches SSTables blocks that were read from GFS.

The Scan Cache is most useful for applications that tend to read the same data repeatedly. The Block Cache is useful for applications that tend to read data that is close to the data they recently read.

### Bloom filters
We reduce the number of accesses by allowing clients to specify that Bloom fil- ters [7] should be created for SSTables in a particu- lar locality group. Our use of Bloom filters also implies that most lookups for non-existent rows or columns do not need to touch disk.

### Speeding up tablet recovery
If the master moves a tablet from one tablet server to another, **the source tablet server first does a minor compaction on that tablet**. This compaction reduces recovery time by reducing the amount of uncompacted state in the tablet server’s commit log. 

### Exploiting immutability
Besides the SSTable caches, various other parts of the Bigtable system have been simplified by the fact that all of the SSTables that we generate are immutable. As a result, concurrency control over rows can be implemented very efficiently. The only mutable data structure that is accessed by both reads and writes is the memtable. To reduce contention during reads of the memtable, we make each memtable row copy-on-write and allow reads and writes to proceed in parallel.

## Google's Lessons
1. large distributed sys- tems are vulnerable to many types of failures, not just the standard network partitions and fail-stop failures as- sumed in many distributed protocols.We added checksumming to our RPC mechanism.
2. It is important to delay adding new features until it is clear how the new features will be used. 
3. The most important lesson we learned is the value of simple designs.



## HBase的实现
HBase是一个开源的非关系型分布式数据库（NoSQL），它参考了谷歌的BigTable建模，实现的编程语言为 Java。但是也有很多不同之处。比如：Google Bigtable利用GFS作为其文件存储系统，HBASE利用Hadoop HDFS作为其文件存储系统；Google运行MAPREDUCE来处理Bigtable中的海量数据，HBASE同样利用Hadoop MapReduce来处理HBASE中的海量数据；Google Bigtable利用Chubby作为协同服务，HBASE利用Zookeeper作为对应。

Rowkey的概念和mysql中的主键是完全一样的，Hbase使用Rowkey来唯一的区分某一行的数据。

由于Hbase只支持3中查询方式：

- 基于Rowkey的单行查询
- 基于Rowkey的范围扫描
- 全表扫描
- 
#### HBase二级索引
每一个索引建立一个表，然后依靠表的row key来实现范围检索。row key在HBase中是以**B+ tree**结构化有序存储的，所以scan起来会比较效率。
单表以row key存储索引，column value存储id值或其他数据 ，这就是Hbase索引表的结构。

### 参考
[HBase二级索引与Join](http://jm.taobao.org/2011/05/29/951/)

[Hbase技术详细学习笔记](https://www.jianshu.com/p/569106a3008f)

[Hbase基础与原理详解](https://blog.csdn.net/ForgetThatNight/article/details/79605829)


