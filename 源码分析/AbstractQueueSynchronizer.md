
# AbstractQueueSynchronizer
## 队列
### Sync Queue
锁实现的数据结构。没有获取锁的线程被设置到一个Node对象，Node对象将通过CAS方式放入到同步队列，该队列是个双向链表结构。。

#### 获取锁
通过acquire方法尝试获取锁，先进行tryAcquire，比如在ReentrantLock中，假如state=0，那么会尝试用cas方法设置state为1，设置成功后进行setExclusiveOwnerThread操作把当前线程记录到aqs结构中，也就是重入锁逻辑的一部分内容。假如tryAcquire失败，会先把当前节点加入到sync queue中，然后进入acquireQueued中，又如果此时当前node的前置node是head节点，那么在此尝试tryAcquire操作，如果不是则进行休眠。

#### 释放锁
release方法中，先进行tryRelease操作，比如在ReentrantLock中，先将state减去1，如果state=0那么再进行setExclusiveOwnerThread(null)操作。然后再唤醒当前节点的后置节点。

### Condition Queue
实现条件队列await与signal的数据结构。

![img](D:\git\study-notes\源码分析\aqs.png)

#### Condition对象的await/signal/signalAll解析
- await过程：先进行加锁操作，然后把当前线程对应的waitStatus为CONDITION类型的node加入到condition queue，然后通过fullyRelease方法释放锁，最后通过park睡眠。
- signal过程：先进行加锁操作，取出condition queue的第一个元素，修改node的waitStatus状态为0，然后加入到sync queue，接着设置前置节点waitStatus为-1，最后进行释放锁的操作，也就是刚刚加入到sync queue的元素将有可能被唤醒。
- signalAll过程：与signal不同的是，遍历condition queue里每个元素，然后执行signal同样的操作。

####  Why Lock condition await must hold the lock
So await must hold the lock because otherwise there would be no way to ensure you weren't waiting for something that already happened. You must hold the lock to prevent another thread from racing with your wait.

## 与Synchronized的区别

`ReentrantLock`显示获得、释放锁，`synchronized`隐式获得释放锁

`ReentrantLock`可响应中断、可轮回，`synchronized`是不可以响应中断的，为处理锁的不可用性提供了更高的灵活性

`ReentrantLock`是`API`级别的，`synchronized`是`JVM`级别的

`ReentrantLock`可以实现公平锁

`ReentrantLock`通过`Condition`可以绑定多个条件

```java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    private static final long serialVersionUID = 3737899427754241961L;

    protected AbstractOwnableSynchronizer() { }

    private transient Thread exclusiveOwnerThread;

    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}

public class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    
    private transient volatile Node head;

    /**
     * Tail of the wait queue, lazily initialized.  Modified only via
     * method enq to add new wait node.
     */
    private transient volatile Node tail;

    /**
     * The synchronization state. 使用volatile因为多线程访问该值时，需要立马知道该值状态，便于后续加锁操作。
     */
    private volatile int state;
    
}


static final class Node {
        /** Marker to indicate a node is waiting in shared mode */
        static final Node SHARED = new Node();
        /** Marker to indicate a node is waiting in exclusive mode */
        static final Node EXCLUSIVE = null;

        /** waitStatus value to indicate thread has cancelled */
        static final int CANCELLED =  1;
        /** waitStatus value to indicate successor's thread needs unparking */
        static final int SIGNAL    = -1;
        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;
        /**
         * waitStatus value to indicate the next acquireShared should
         * unconditionally propagate
         */
        static final int PROPAGATE = -3;

        /**
         * Status field, taking on only the values:
         *   SIGNAL:     The successor of this node is (or will soon be)
         *               blocked (via park), so the current node must
         *               unpark its successor when it releases or
         *               cancels. To avoid races, acquire methods must
         *               first indicate they need a signal,
         *               then retry the atomic acquire, and then,
         *               on failure, block.
         *   CANCELLED:  This node is cancelled due to timeout or interrupt.
         *               Nodes never leave this state. In particular,
         *               a thread with cancelled node never again blocks.
         *   CONDITION:  This node is currently on a condition queue.
         *               It will not be used as a sync queue node
         *               until transferred, at which time the status
         *               will be set to 0. (Use of this value here has
         *               nothing to do with the other uses of the
         *               field, but simplifies mechanics.)
         *   PROPAGATE:  A releaseShared should be propagated to other
         *               nodes. This is set (for head node only) in
         *               doReleaseShared to ensure propagation
         *               continues, even if other operations have
         *               since intervened.
         *   0:          None of the above
         *
         * The values are arranged numerically to simplify use.
         * Non-negative values mean that a node doesn't need to
         * signal. So, most code doesn't need to check for particular
         * values, just for sign.
         *
         * The field is initialized to 0 for normal sync nodes, and
         * CONDITION for condition nodes.  It is modified using CAS
         * (or when possible, unconditional volatile writes).
         */
        volatile int waitStatus;

        volatile Node prev;

        volatile Node next;

        /**
         * The thread that enqueued this node.  Initialized on
         * construction and nulled out after use.
         */
        volatile Thread thread;

        /**
         * Link to next node waiting on condition, or the special
         * value SHARED.  Because condition queues are accessed only
         * when holding in exclusive mode, we just need a simple
         * linked queue to hold nodes while they are waiting on
         * conditions. They are then transferred to the queue to
         * re-acquire. And because conditions can only be exclusive,
         * we save a field by using special value to indicate shared
         * mode.
         */
        Node nextWaiter;

        /**
         * Returns true if node is waiting in shared mode.
         */
        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        /**
         * Returns previous node, or throws NullPointerException if null.
         * Use when predecessor cannot be null.  The null check could
         * be elided, but is present to help the VM.
         *
         * @return the predecessor of this node
         */
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
    
    
public final void acquire(int arg) {
//try acquire子类实现尝试获取锁操作
    if (!tryAcquire(arg) &&  
//tryAcquire失败，则将当前线程CAS方式加入到队列，假如当前线程发现加入等待队列后，自己为第一个元素，则会再次尝试tryAcquire操作，若失败，则会设置前节点既head节点的waitStatus为SIGNAL，然后假如自己还是队列第一个节点，再次尝试（第一个节点进行自旋一次），如果此次还是失败，则进行线程休眠。
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) 
        selfInterrupt();
}

/**
* 以独占的方式获取。
* 子类中实现。参数arg，由子类自定义，一般情况下都需要CAS方式进行设置，因为独占锁只被一个线程占有。
*
*/
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}

/**
 * Acquires in exclusive uninterruptible mode for thread already in
 * queue. Used by condition wait methods as well as acquire.
 *
 * @param node the node
 * @param arg the acquire argument
 * @return {@code true} if interrupted while waiting
 */
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            //如果p等于head头结点，那么说明该元素此时是第一个等待元素，那么可以再次尝试获取锁操作。
            if (p == head && tryAcquire(arg)) { 
                /* 进行替换，因为此时有可能有等待线程在队列，需要通过release进行唤醒，release唤醒的话需要head的waitStatus不等于0。
                这里有可能出现head的waitStatus不为0，比如加入到队列元素只有一个时候，也恰巧，同时tryAcquire成功，那么此时的head.waitStatus也为0。*/
                setHead(node); //争抢成功后，说明当前线程已经加锁，所以不用加入到等待队列。
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}


/**
 * Checks and updates status for a node that failed to acquire.
 * Returns true if thread should block. This is the main signal
 * control in all acquire loops.  Requires that pred == node.prev.
 *
 * @param pred node's predecessor holding status
 * @param node the node
 * @return {@code true} if thread should block
 */
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        /*
         * This node has already set status asking a release
         * to signal it, so it can safely park.
         */
        return true;
    if (ws > 0) {
        /*
         * Predecessor was cancelled. Skip over predecessors and
         * indicate retry.
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else { //此时ws等于0，也就是head初始结点，因为有下一个等待线程结点，需要设置head结点的waitStatus为SIGNAL，表明有等待结点。
        /*
         * waitStatus must be 0 or PROPAGATE.  Indicate that we
         * need a signal, but don't park yet.  Caller will need to
         * retry to make sure it cannot acquire before parking.
         */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}

/**
 * 通过CAS方式，加入到队尾，也就是加到tail后面。
 * Creates and enqueues node for current thread and given mode.
 * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
 * @return the new node
 */
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}

/**
 * 这个方法其中一次被addWaiter使用，主要是防止在addWait方法中CAS加队尾失败，再
 * 次在此方法里重试。
 * Inserts node into queue, initializing if necessary. See picture above.
 * @param node the node to insert
 * @return node's predecessor
 */
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize 初始化head为一个头结点（仅仅是占位），接着循环一次再设置tail
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}

public final boolean release(int arg) {
    if (tryRelease(arg)) {//一定是当前线程进行解锁才是正常，state减arg
        Node h = head;
        //假如是null，说明队列为空，如果waitStatus=0，说明没有等待元素，因为一旦有等待结点，那么一定head会被替换成等待的结点，具体逻辑在acquireQueue里
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

/**
 * Wakes up node's successor, if one exists.
 *
 * @param node the node
 */
private void unparkSuccessor(Node node) {
    /* 设置头结点的waitStatus为0，目的是为了让线程有机会再次tryAcquire操作，逻辑见acquireQueued
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /* 取到头结点的下一个线程结点，如果不为空并且不是CANCELED，则唤醒该结点。
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread); //唤醒头结点的下一个等待的线程结点
}

public final void acquireShared(int arg) {
//在ReentrantReadWriteLock如果当前已经有写锁，或者写锁位于队列第一个节点时，则尝试多给写锁机会。
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

 private void doAcquireShared(int arg) {
       ...
    for (;;) {
        final Node p = node.predecessor();
        if (p == head) {
//在ReentrantReadWriteLock中，获取成功成功，返回值>0。
            int r = tryAcquireShared(arg); 
            if (r >= 0) { 
//逻辑有isShared()=true的node进行doReleaseShared操作。
                setHeadAndPropagate(node, r);
       ...
}

/**
* 是AbstractQueueSynchronizer的内部类，也就是可以新建此ConditionObject时，该类可以直接访问AbstractQueueSynchronizer内部属性。
*/
public class ConditionObject implements Condition, java.io.Serializable {
    private static final long serialVersionUID = 1173984872572414699L;
    // 以下两个头结点和尾结点表明该条件对象里有一个队列
    /** First node of condition queue. */
    private transient Node firstWaiter;
    /** Last node of condition queue. */
    private transient Node lastWaiter;

    /**
     * Creates a new {@code ConditionObject} instance.
     */
    public ConditionObject() { }
    
    /**
    * Adds a new waiter to wait queue.
    * @return its new wait node
    */
    private Node addConditionWaiter() {
        Node t = lastWaiter;
        // If lastWaiter is cancelled, clean out.
        if (t != null && t.waitStatus != Node.CONDITION) {
            unlinkCancelledWaiters();
            t = lastWaiter;
        }
        Node node = new Node(Thread.currentThread(), Node.CONDITION);
        if (t == null)
            firstWaiter = node;
        else
            t.nextWaiter = node;
        lastWaiter = node;
        return node;
    }

    public final void await() throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
      //把当前线程对应waitStatus为CONDITION的node加入到condition queue
        Node node = addConditionWaiter();
      //进入await方法前需要加锁，此时当前线程已经加入到condition queue，那么需要释放锁。
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
      //当前线程不在sync queue，说明没有被signal进入sync queue，则进行休眠。
            LockSupport.park(this);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }
    
    public final void signal() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
        //发起信号给第一个节点
            doSignal(first);
    }
    
    private void doSignal(Node first) {
        do {
            if ( (firstWaiter = first.nextWaiter) == null)
                lastWaiter = null;
            first.nextWaiter = null;
        } while (!transferForSignal(first) &&
                 (first = firstWaiter) != null);
    }
    /**
     * Removes and transfers all nodes.
     * @param first (non-null) the first node on condition queue
     */
    private void doSignalAll(Node first) {
        lastWaiter = firstWaiter = null;
        do {
            Node next = first.nextWaiter;
            first.nextWaiter = null;
            transferForSignal(first);
            first = next;
        } while (first != null);
    }

}

//把当前node从condition queue中放入到sync queue中，等待该结点对应线程获取锁后被唤醒
final boolean transferForSignal(Node node) {
    /*
     * If cannot change waitStatus, the node has been cancelled.
     */
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    /*
     * Splice onto queue and try to set waitStatus of predecessor to
     * indicate that thread is (probably) waiting. If cancelled or
     * attempt to set waitStatus fails, wake up to resync (in which
     * case the waitStatus can be transiently and harmlessly wrong).
     * 加入到sync queue队列
     */
    Node p = enq(node);
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL)) 
    //大部分情况下不会进入此方法，也就前置节点状态不是cancel或不会设置signal失败
        LockSupport.unpark(node.thread);
    return true;
}


```

## 例子

```java
public class ConditionMain {

    private Lock lock = new ReentrantLock();

    private Condition condition1 = lock.newCondition();

    private Condition condition2 = lock.newCondition();

    private boolean isAvailable = true;
    public void lock(){
        try{
            lock.lock();
            if(isAvailable){
                try {
                    System.out.println("lock await");
                    condition1.await();
                    System.out.println("await1 end");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("lock end");
            Thread.sleep(1000);

        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }
    public void lock2(){
        try{
            lock.lock();
            if(isAvailable){
                try {
                    System.out.println("lock2 await");
                    condition2.await();
                    System.out.println("await2 end");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("lock2 end");
            Thread.sleep(1000);

        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    public void release1(){
        try{
            System.out.println("release1 enter");
            lock.lock();
            isAvailable = true;
            condition1.signalAll();
            System.out.println("signalAll 1");
            Thread.sleep(1000);
            System.out.println("release1 sleep");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void release2(){
        try{
            System.out.println("release2 enter");
            lock.lock();
            isAvailable = true;
            condition2.signalAll();
            System.out.println("signalAll 2");
            Thread.sleep(1000);
            System.out.println("release2 sleep");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args){
        final  ConditionMain object = new ConditionMain();
        new Thread(new Runnable() {
            @Override
            public void run() {
                object.lock();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                object.lock2();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                object.release1();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                object.release2();
            }
        }).start();

    }
}
```

