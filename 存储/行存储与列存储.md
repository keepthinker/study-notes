# 行存储简介
A column-oriented DBMS (or columnar database management system) is a database management system (DBMS) that stores data tables by column rather than by row. By storing data in columns rather than rows, the database can more precisely access the data it needs to answer a query rather than scanning and discarding unwanted data in rows. Query performance is increased for certain workloads.

# Row-oriented systems
A common method of storing a table is to serialize each row of data, like this:
```
001:10,Smith,Joe,40000;
002:12,Jones,Mary,50000;
003:11,Johnson,Cathy,44000;
004:22,Jones,Bob,55000;
```
As data is inserted into the table, it is assigned an internal ID, the rowid that is used internally in the system to refer to data. In this case the records have sequential rowids independent of the user-assigned empid. In this example, the DBMS uses short integers to store rowids. In practice, larger numbers, 64-bit or 128-bit, are normally used.

Row-based systems are designed to efficiently return data for an entire row, or record, in as few operations as possible. This matches the common use-case where the system is attempting to retrieve information about a particular object. By storing the record's data in a single block on the disk, along with related records, the system can quickly retrieve records with a minimum of disk operations.

## Advantage of row-based system
Operations that retrieve all the data for a given object (the entire row) are slower. A row-based system can retrieve the row in a single disk read, whereas numerous disk operations to collect data from multiple columns are required from a columnar database.

行存储的写入是一次性完成，消耗的时间比列存储少，并且能够保证数据的完整性.

## Disadvantage of row-based system
Row-oriented databases are well-suited for OLTP-like workloads which are more heavily loaded with interactive transactions. For example, retrieving all data from a single row is more efficient when that data is located in a single location (minimizing disk seeks), as in row-oriented architectures. 

数据读取过程中会产生冗余数据，如果只有少量数据，此影响可以忽略；数量大可能会影响到数据的处理效率

# Column-oriented systems
A column-oriented database serializes all of the values of a column together, then the values of the next column, and so on. For our example table, the data would be stored in this fashion:
```
10:001,12:002,11:003,22:004;
Smith:001,Jones:002,Johnson:003,Jones:004;
Joe:001,Mary:002,Cathy:003,Bob:004;
40000:001,50000:002,44000:003,55000:004;
```
In this layout, any one of the columns more closely matches the structure of an index in a row-based system. 

## 改进
减少冗余数据首先是用户在定义数据时避免冗余列的产生；其次是优化数据存储记录结构，保证从磁盘读出的数据进入内存后，能够被快速分解，消除冗余列。要知道，目前市场上即使最低端 CPU 和内存的速度也比机械磁盘快上 100-1000 倍。如果用上高端的硬件配置，这个处理过程还要更快。

## Advantage of column-based system
Whole-row operations are generally rare. In the majority of cases, **only a limited subset of data is retrieved**. In a rolodex application, for instance, collecting the first and last names from many rows to build a list of contacts is far more common than reading all data for any single address. This is even more true for writing data into the database, especially if the data tends to be **"sparse" with many optional columns**. For this reason, column stores have demonstrated excellent real-world performance in spite of many theoretical disadvantages.

Columnar databases boost performance by reducing the amount of data that needs to be read from disk, both by efficiently **compressing the similar columnar data** and by **reading only the data necessary** to answer the query.

Columnar databases are well-suited for OLAP-like workloads (e.g., data warehouses) which typically involve highly complex queries over all data (possibly petabytes).

由于列存储的每一列数据类型是同质的，不存在二义性问题。比如说某列数据类型为整型（int），那么它的数据集合一定是整型数据。这种情况使数据解析变得十分容易。相比之下，行存储则要复杂得多，因为在一行记录中保存了多种类型的数据，数据解析需要在多种数据类型之间频繁转换，这个操作很消耗 CPU，增加了解析的时间。所以，列存储的解析过程更有利于分析大数据。

## Disadvantage of column-based system
Transactions (INSERTs) must be separated into columns and compressed as they are stored, making it less suited for OLTP workloads.

列存储由于需要把一行记录拆分成单列保存，写入次数明显比行存储多，再加上磁头需要在盘片上移动和定位花费的时间，实际时间消耗会更大。

## Compression
Column data is of uniform type; therefore, there are some opportunities for storage size optimizations available in column-oriented data that are not available in row-oriented data.

Columnar compression achieves a reduction in disk space at the expense of efficiency of retrieval. The greater adjacent compression achieved, the more **difficult random-access may become**, as data might need to be uncompressed to be read.

## 改进
1. 在计算机上安装多块硬盘，以多线程并行的方式读写它们。多块硬盘并行工作可以减少磁盘读写竞用，这种方式对提高处理效率优势十分明显。缺点是需要更多的硬盘，这会增加投入成本，在大规模数据处理应用中是不小的数目，运营商需要认真考虑这个问题。
2. 对写过程中的数据完整性问题，可考虑在写入过程中加入类似关系数据库的“回滚”机制，当某一列发生写入失败时，此前写入的数据全部失效，同时加入散列码校验，进一步保证数据完整性。

# 两种存储方案的共同改进
频繁的小量的数据写入对磁盘影响很大，更好的解决办法是将数据在内存中暂时保存并整理，达到一定数量后，一次性写入磁盘，这样消耗时间更少一些。目前机械磁盘的写入速度在 20M-50M/ 秒之间，能够以批量的方式写入磁盘，效果也是不错的。

# 参考文献
[Column-oriented DBMS](https://en.wikipedia.org/wiki/Column-oriented_DBMS)

[大数据存取的选择：行存储还是列存储？](https://www.infoq.cn/article/bigdata-store-choose)