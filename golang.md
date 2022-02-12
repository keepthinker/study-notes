# Golang

## Coroutine

**Coroutines** are computer program components that generalize subroutines for non-preemptive multitasking, by allowing execution to be suspended and resumed. Coroutines are well-suited for implementing familiar program components such as cooperative tasks, exceptions, event loops, iterators, infinite lists and pipes.

According to Donald Knuth, Melvin Conway coined the term *coroutine* in 1958 when he applied it to the construction of an assembly program. The first published explanation of the coroutine appeared later, in 1963.

### Comparison with threads

Coroutines are very similar to threads. However, coroutines are cooperatively multitasked, whereas threads are typically preemptively multitasked. This means that coroutines provide concurrency but not parallelism. The advantages of coroutines over threads are that they may be used in a hard-realtime context switching between coroutines need not involve any system calls or any blocking calls whatsoever, there is no need for synchronisation primitives such as mutexes semaphores, etc. in order to guard critical sections, and there is no need for support from the operating system.

It is possible to implement coroutines using preemptively-scheduled threads, in a way that will be transparent to the calling code, but some of the advantages (particularly the suitability for hard-realtime operation and relative cheapness of switching between them) will be lost.

我们知道操作系统在线程等待IO的时候，会阻塞当前线程，切换到其它线程，这样在当前线程等待IO的过程中，其它线程可以继续执行。当系统线程较少的时候没有什么问题，但是当线程数量非常多的时候，却产生了问题。**一是系统线程会占用非常多的内存空间，二是过多的线程切换会占用大量的系统时间。**

协程刚好可以解决上述2个问题。协程运行在线程之上，当一个协程执行完成后，可以选择主动让出，让另一个协程运行在当前线程之上。**协程并没有增加线程数量，只是在线程的基础之上通过分时复用的方式运行多个协程**，而且协程的切换在用户态完成，切换的代价比线程从用户态到内核态的代价小很多。

### 注意事项

假设协程运行在线程之上，并且协程调用了一个阻塞IO操作，这时候会发生什么？实际上操作系统并不知道协程的存在，它只知道线程，**因此在协程调用阻塞IO操作的时候，操作系统会让线程进入阻塞状态，当前的协程和其它绑定在该线程之上的协程都会陷入阻塞而得不到调度，这往往是不能接受的。**

因此在协程中不能调用导致线程阻塞的操作。也就是说，协程只有和异步IO结合起来，才能发挥最大的威力，该用异步调用方式，需要大量工作，最好依赖语言实现本身。

协程对计算密集型的任务也没有太大的好处，计算密集型的任务本身不需要大量的线程切换，因此协程的作用也十分有限，反而还增加了协程切换的开销。

在有大量IO操作业务的情况下，我们采用协程替换线程，可以到达很好的效果，一是降低了系统内存，二是减少了系统切换开销，因此系统的性能也会提升。

在协程中尽量不要调用阻塞IO的方法，比如打印，读取文件，Socket接口等，除非改为异步调用的方式，并且协程只有在IO密集型的任务中才会发挥作用。

**协程只有和异步IO结合起来才能发挥出最大的威力。**

### 参考文献

[什么是协程？](https://zhuanlan.zhihu.com/p/172471249)

## golang 的 gc 算法

以下是Golang GC算法的里程碑：

- v1.1 STW
- v1.3 Mark STW, Sweep 并行
- v1.5 三色标记法
- v1.8 hybrid write barrier

经典的GC算法有三种：`引用计数(reference counting)`、`标记-清扫(mark & sweep)`、`复制收集(Copy and Collection)`。

Golang的GC算法主要是基于`标记-清扫(mark and sweep)`算法，并在此基础上做了改进。

### 标记-清扫(Mark And Sweep)算法

此算法主要有两个主要的步骤：

- 标记(Mark phase)
- 清除(Sweep phase)

第一步，找出可达的对象，然后做上标记。 第二步，回收未标记的对象。

操作非常简单，但是有一点需要额外注意：mark and sweep算法在执行的时候，需要程序暂停！即`stop the world`。 也就是说，这段时间程序会卡在哪儿。

#### 标记-清扫(Mark And Sweep)算法存在什么问题？

标记-清扫(Mark And Sweep)算法这种算法虽然非常的简单，但是还存在一些问题：

- STW，stop the world；让程序暂停，程序出现卡顿。
- 标记需要扫描整个heap
- 清除数据会产生heap碎片

这里面最重要的问题就是：mark-and-sweep 算法会暂停整个程序。

### 三色并发标记法

1. 首先，程序创建的对象都标记为白色。

2. gc开始，扫描所有可到达的对象，标记为灰色

3. 从灰色对象中找到其引用对象标记为灰色，把灰色对象本身标记为黑色

4. 监视对象中的内存修改，并持续上一步的操作，直到灰色标记的对象不存在

5. 此时，gc回收白色对象。

6. 最后，将所有黑色对象变为白色，并重复以上所有过程。

7. 回收白色对象时经过写内存屏障可监视白色对象是否被修改，如果被修改那么进行重新标记。

标记-清除(mark and sweep)算法的STW(stop the world)操作，就是runtime把所有的线程全部冻结掉，所有的线程全部冻结意味着用户逻辑是暂停的。这样所有的对象都不会被修改了，这时候去扫描是绝对安全的。

Go如何减短这个过程呢？标记-清除(mark and sweep)算法包含两部分逻辑：标记和清除。 我们知道Golang三色标记法中最后只剩下的黑白两种对象，黑色对象是程序恢复后接着使用的对象，如果不碰触黑色对象，只清除白色的对象，肯定不会影响程序逻辑。所以：`清除操作和用户逻辑可以并发。

#### gc和用户逻辑如何并行操作？

process新生成对象的时候，GC该如何操作呢？不会乱吗？Golang为了解决这个问题，引入了写屏障这个机制。 写屏障：该屏障之前的写操作和之后的写操作相比，先被系统其它组件感知。 通俗的讲：就是在gc跑的过程中，可以监控对象的内存修改，并对对象进行重新标记。(实际上也是超短暂的stw，然后对对象进行标记)

The primary principle behind the tricolor mark-and-sweep algorithm is that it divides the objects of the heap into three different sets according to their color, which is assigned by the algorithm. It is now time to talk about the meaning of each color set. 

1. The objects of the **black set** are guaranteed to have no pointers to any object of the white set. 
2. However, an object of the **white set** can have a pointer to an object of the black set because this has no effect on the operation of the garbage collector. 
3. The objects of the **gray set** might have pointers to some objects of the white set. 
4. Finally, the objects of the **white set** are the candidates for garbage collection.

The **roots** are the objects that can be directly accessed by the application, which includes **global variables and other things on the stack**. These objects mostly depend on the Go code of a program.

#### Write Barrier

During this process, the running application is called the **mutator**. The mutator runs a small function named **write barrier** that is executed each time a pointer in the heap is modified. If the pointer of an object in the heap is modified, which means that this object is now reachable, the write barrier colors it gray and puts it in the gray set.

Garbage Collectors in Go are either optimized for **lower latency** or **higher throughput**. They might also perform better or worse at these depending on the heap usage of your program. Tricolor mark-and-sweep algorithm is used in Go to lower that latency by running the garbage collector as a concurrent process.

### 参考文献

[图解Golang的GC算法](https://juejin.cn/post/6844903793855987719)

[Implementing memory management with Golang’s garbage collector](https://hub.packtpub.com/implementing-memory-management-with-golang-garbage-collector/)

## kubernetes 的架构
