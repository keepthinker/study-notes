# 基础支持

## LockSupport

```java
/**Disables the current thread for thread scheduling purposes unless the permit is available.
   @param blocker the synchronization object responsible for this thread parking
 */
public static void park(Object blocker);

/**
Disables the current thread for thread scheduling purposes, for up to the specified waiting time, unless the permit is available.
*/
public static void parkNanos(Object blocker, long nanos);

/**
Disables the current thread for thread scheduling purposes unless the permit is available.
*/
public static void park();
/**
Makes available the permit for the given thread, if it was not already available.  If the thread was blocked on
{@code park} then it will unblock.  Otherwise, its next call to {@code park} is guaranteed not to block. This operation
is not guaranteed to have any effect at all if the given thread has not been started.
*/
public static void unpark(Thread thread)
```

## Unsafe cas方法

```java
//得到指定对对象成员变量对应的long值，为CAS操作做指引。
public native long objectFieldOffset(Field var1);
//例子
stateOffset = unsafe.objectFieldOffset(AbstractQueuedSynchronizer.class.getDeclaredField("state"));

//进行cas操作，如果var1对象的var2所在成员变量等于var4，那么将会把var5替换var4。
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);
```

## 条件队列

线程间互相唤醒对方。

### Object.wait() & Object.notify()

```java
public final native void notify();
public final native void wait(long timeout) throws InterruptedException;
```

- 与synchronized配套使用。
- 线程执行wait前需要synchronized加锁，执行wait后，将会释放锁。
- 线程执行notify前需要synchronized加锁，执行notify后，需要跳出synchronized作用域解锁，才能唤醒被wait的对象。

### Condition.await() & Condition.signal()

- 域继承Lock接口的对象使用，比如ReentrantLock。
- 执行这两个函数时雨wait/notify/notifyAll类似。

## volatile

### 可见性

可见性是指多个线程访问同一个变量时，其中一个线程修改了该变量的值，其它线程能够立即看到修改的值。

Java内存模型是通过在变量修改后将新值从工作内存同步回主内存，在变量读取前从主内存刷新变量值这种依赖主内存作为传递媒介的方式来实现可见性的，无论是普通变量还是volatile变量都是如此，普通变量与volatile变量的区别是，**volatile的特殊规则保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新到工作内存**。因此，可以说volatile保证了多线程操作时变量的可见性，而普通变量则不能保证这一点。

除了volatile之外，Java还有两个关键字能实现可见性，即synchronized和final。

### 有序性

一个线程中的所有操作必须按照程序的顺序来执行。

volatile修饰的变量不会被指令重排序优化，保证代码的执行顺序与程序的顺序相同。为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入**内存屏障**来禁止重排序。

### 参考文献

[volatile关键字详解](https://zhuanlan.zhihu.com/p/34362413)

# 并发控制类

## Lock

```java
//获取不到锁则进行阻塞。
void lock();
//如果实现类支持的话，允许其他线程interrupt当前等待锁而睡眠的线程。
void lockInterruptibly()
//取到锁直接返回true，取不到锁则返回false。不会阻塞当前线程。
boolean tryLock();
//在time时间内取到锁直接返回，如果在time时间之后还取不到锁则返回false。
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
void unlock();
//生产针对该锁的一个条件队列。
Condition newCondition();
```

## ReentrantLock

可重入锁。支持公平锁与非公平锁。

## ReadWriteLock

允许多个读操作同时进行，但每次只允许一个写操作，也就是说写锁是独占的。当然假如现在代码块被读锁占用，那么写锁需要等待读锁释放后来竞争。

## CoundownLatch

初始化设置count值，表明release count次，才会唤醒被await的所有线程。

使用共享模式，因为await方法可以被多个线程使用，当countdown使state的值归0时，将唤醒head下一个node关联的线程争抢锁，然后通过setHeadAndPropagate设置完当前线程的head节点后，唤醒下一个节点的线程，当然下一个节点又是head的下一个node，故将进行上诉操作进行释放锁。。

```java
//初始化中count值实际上设置到AQS中的state属性
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

## Cyclibarrier

所有被await的线程数达到类初始化的参数时，将会先执行初始化中的Runnable然后再，将所有await线程唤醒。

在该类中，有ReentrantLock，Condition对象字段，一个是进行加锁操作，一个为了实现休眠与唤醒逻辑。

```java
/**Main barrier code, covering the various policies. */
    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        final ReentrantLock lock = this.lock;
        lock.lock(); //先进行加锁，为了保证index被原子递减
        try {
            ...
            int index = --count;
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    if (command != null)
                        command.run();
                    ranAction = true;
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }

            // loop until tripped, broken, interrupted, or timed out
            for (;;) {
                try {//如果index不为0，则进行睡眠操作
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        nanos = trip.awaitNanos(nanos);
               ...

            }
        } finally {
            lock.unlock();
        }
    }
```

## Sephemore

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = 2014338818796000944L;

   //将permits值设置到state
    FairSync(int permits) {
        super(permits);
    }

    //假如剩余的permits数大于0，则当前线程继续执行下去，否则进入sync queue队列并进行睡眠操作。
    protected int tryAcquireShared(int acquires) {
        for (;;) { //循环进行cas操作，直到设置成功
            if (hasQueuedPredecessors())
                return -1;
            int available = getState();
            //每次调用acquire()，则将可用的剩余的state既permits数减1，这里acquires变量在Semaphore里是1
            int remaining = available - acquires;
           //假如permits数小于0，则说明semaphore已经用尽，那么当前线程需要进行休眠，等待被唤醒
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
}

abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 1192457210091910933L;

    Sync(int permits) {
        setState(permits);
    }

    // tryReleaseShared成功，则唤醒sync queue中睡眠的线程
    protected final boolean tryReleaseShared(int releases) {
        for (;;) { //循环进行cas操作，直到设置成功
            int current = getState();
            //在semaphore里release是1，每一次release()操作相当于增加一个permit
            int next = current + releases;
            if (next < current) // overflow
                throw new Error("Maximum permit count exceeded");
            if (compareAndSetState(current, next))
                return true;
        }
    }
```

## ThreadPoolExecutor

继承AbstractExecutorService，实现ExecutorService接口，其中通过ctl(AtomicInteger)来控制线程池状态与当前工作任务数。如果工作线程小于corePoolSize则直接运行，如果大于corePoolSize并小于workQueue的容量，则放入workQueue，否则多开启maxPoolSize - corePoolSize数量的工作线程。

runWorker(Worker w)为核心线程池运行逻辑。如果当前获取到的task为null并且workQueue为空，则根据情况采用定时轮询时间检查是否丢弃超过corePoolSize的线程，若线程回归到corePoolSize则进行调用workQueue.take()，若取不到元素则阻塞等待。

### AbstractExecutorService

实现基本的提交任务方法，比如submit(Runnable task),  submit(Callable<T> task)。实际上所有的提交的任务都被封装到RunnableFuture对象，然后再执行线程任务。

### 优先级线程池

由于PriorityBlockingQueue线程需要里面元素进行比较需要元素实现Comparable接口，故ThreadPoolExecutor线程池中的workQueue添加对象既实现Runnable接口的对象也要求实现Comparable接口。

# Thread

## void yield()

A hint to the scheduler that the current thread is willing to yield its current use of a processor. The scheduler is free to ignore this
hint.

## void sleep(long millis) throws InterruptedException

Causes the currently executing thread to sleep (temporarily cease execution) for the specified number of milliseconds, subject to the precision and accuracy of system timers and schedulers. The thread does not lose ownership of any monitors.

## void start()

Causes this thread to begin execution; the Java Virtual Machine
calls the <code>run</code> method of this thread.

The result is that two threads are running concurrently: the current thread (which returns from the call to the
<code>start</code> method) and the other thread (which executes its <code>run</code> method).

It is never legal to start a thread more than once. In particular, a thread may not be restarted once it has completed execution.

## void interrupt()

Interrupts this thread.
Unless the current thread is interrupting itself, which is
always permitted, the {@link #checkAccess() checkAccess} method
of this thread is invoked, which may cause a {@link
SecurityException} to be thrown.

If this thread is blocked in an invocation of the {@link
Object#wait() wait()}, {@link Object#wait(long) wait(long)}, or {@link
Object#wait(long, int) wait(long, int)} methods of the {@link Object}
class, or of the {@link #join()}, {@link #join(long)}, {@link
#join(long, int)}, {@link #sleep(long)}, or {@link #sleep(long, int)},
methods of this class, then its interrupt status will be cleared and it
will receive an {@link InterruptedException}.

If this thread is blocked in an I/O operation upon an {@link
java.nio.channels.InterruptibleChannel InterruptibleChannel}
then the channel will be closed, the thread's interrupt
status will be set, and the thread will receive a {@link
java.nio.channels.ClosedByInterruptException}.

If this thread is blocked in a {@link java.nio.channels.Selector}
then the thread's interrupt status will be set and it will return
immediately from the selection operation, possibly with a non-zero
value, just as if the selector's {@link
java.nio.channels.Selector#wakeup wakeup} method were invoked.

If none of the previous conditions hold then this thread's interrupt
status will be set.

Interrupting a thread that is not alive need not have any effect.

# Future

A Future represents the result of an asynchronous computation. Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation

RunnableFuture继承Future与Runnable，也既Runnable执行完run方法后会触发Future事件。

## FutureTask

实现RunnableFuture接口，run方法中会执行set()方法，set()方法将会unpark方式唤醒被get()所阻塞的所有线程。被get()阻塞的线程将通过CAS方式进入waiters队列，假如此时run还没执行完，则执行park()方法阻塞当前线程，释放CPU资源。




## ThreadLocal





# Stream Api

...