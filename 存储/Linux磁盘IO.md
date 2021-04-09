## address_space的内存管理

page cache中的一个page由若干buffer组成，一个buffer等于文件系统中一个block的大小，加入buffer的概念是为了方便写磁盘时将page转换为block。具体结构如下图所示。

<pre>
           page-cache(radix)
          /     |     \
         /      |      \
       page1  page2   page3
       /    
      /             ...
buffer(block大小)     ...
    /  \               ...
   /    \               ...
sector1  sector2         ...
</pre>

每个inode都有一个struct buffer_head链表（inode->i_data.private_list），记录了这个inode里产生的page cache里的所有buffer，每个buffer_head元素的b_page指向该buffer位于redix中的哪个page，而b_data成员就是该buffer在page中的位置的指针。我们在内核中就可以通过sb_bread(sb, block_no)得到文件系统中block_no块的数据了。

所以buffer是page cache里面更小的单元，即page cache的redix树记录所有page，每个page又由多个buffer组成。

虽然上层文件系统文件的page cache已经是块设备之上的东西了，但它的page里面仍分成一个个的buffer，一方面文件系统和块设备文件的address_space结构一致方便管理，另外page cache在回写时就不用做额外转换，块设备层可以直接识别其结构并使用。



## 直接IO（Direct IO）的流程

那么Direct IO就是不经过address_space（即page cache）而是直接读写硬盘，所以Direct IO是要求扇区对齐的，避免page和block的转换。
不经过page cache的方法有两种：
**O_DIRECT**：应用程序直接操作硬盘，不产生page cache。一般用于有用户在用户态自己搞cache的情况（例如数据库缓存）。
**O_SYNC**：写透（write through）模式，应用程序写page cache的同时也写硬盘。所以会立即更新硬盘，但也产生page cache。



## free命令中的cached/buffers

我们看free命令中有cached何buffers，现在这两个已经合并了，也就是上面文章中刚刚说到的“合并”。如果非要细分的话，我们访问磁盘时，如果通过文件系统层打开文件（open(“/home/test.txt”, …)），读写文件时inode产生的page cache在free命令中算在“cached”中，而直接操作裸分区（open(“/dev/sda”, …)或直接操作分区）的读写操作产生的page cache在free命令中算在“buffers”中。

## 用户发起IO行为时，数据的走向

在读写文件时，读写操作只和page cache打交道就好了，例如进程的写只是把数据放到page cache并标记dirty。那么page cache中的数据真正写到磁盘的过程是怎样的呢？



### block和page

文件系统里的管理单元是block，内存管理是以page为单位，而内核中读写磁盘是通过page cache的，所以就要了解bio是怎么将page和block对应起来的。例如，如果文件系统的一个page大小是4K，格式化文件系统时指定一个block是4K，那一个block就对应一个page，而如果一个block大小是1K，那一个page就对应4个block，那每次读磁盘就只能每次最小读4个block的大小。
sector（扇区）是硬件读写的最小操作单元，例如如果一个sector是512字节，而格式化文件系统时指定一个block是1K，那文件系统层面只能每次两个sector一起操作。block是文件系统格式化时确定的，sector是由你的硬盘决定的。但是如果你的block越大，那么你的空间浪费可能就会越大（内部碎片，一个文件的末尾占的block的空闲部分）。

### BIO

即Block IO。就是管硬盘的哪些blocks要读到内存的哪些pages。把inode读到内存时，磁盘里这个inode表里记录了这个inode的数据位于磁盘的哪些blocks，当然这些block在硬盘中可能不连续存放，例如要读出一个inode的32KB的内容，但这些block的分布在4个不连续的位置（b1…b2…b3b4b5b6…b7b8），那readpages会将每一段连续的blocks转化成一个bio结构，每个bio结构都记录它里面的blocks在磁盘里的位置以及这些blocks要填到page cache的哪些位置。例如这里就是要生成4个bio对象。那么在一次读过程中，即使一个block大小是1KB，但由于它和其他blocks不连续，那也要单独生成一个bio。



## 参考文献

[Linux IO 学习笔记（二）——文件系统读写文件的流程](https://blog.csdn.net/frecon/article/details/80153535)

[IO/内存/文件系统](