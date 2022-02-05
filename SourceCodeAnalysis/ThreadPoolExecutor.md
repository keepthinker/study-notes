# ThreadPoolExecutor
继承AbstractExecutorService，实现ExecutorService接口，其中通过ctl(AtomicInteger)来控制线程池状态与当前工作任务数。如果工作线程小于corePoolSize则直接运行，如果大于corePoolSize并小于workQueue的容量，则放入workQueue，否则多开启maxPoolSize - corePoolSize数量的工作线程。

runWorker(Worker w)为核心线程池运行逻辑。如果当前获取到的task为null并且workQueue为空，则根据情况采用定时轮询时间检查是否丢弃超过corePoolSize的线程，若线程回归到corePoolSize则进行调用workQueue.take()，若取不到元素则阻塞等待。

```java


    // 主要是对workers的操作进行加锁保护
    private final ReentrantLock mainLock = new ReentrantLock();
    /*
     * RUNNING -> SHUTDOWN
     *    On invocation of shutdown(), perhaps implicitly in finalize()
     * (RUNNING or SHUTDOWN) -> STOP
     *    On invocation of shutdownNow()
     * SHUTDOWN -> TIDYING
     *    When both queue and pool are empty
     * STOP -> TIDYING
     *    When pool is empty
     * TIDYING -> TERMINATED
     *    When the terminated() hook method has completed
     *
     * Threads waiting in awaitTermination() will return when the
     * state reaches TERMINATED.
     *
     * Detecting the transition from SHUTDOWN to TIDYING is less
     * straightforward than you'd like because the queue may become
     * empty after non-empty and vice versa during SHUTDOWN state, but
     * we can only terminate if, after seeing that it is empty, we see
     * that workerCount is 0 (which sometimes entails a recheck -- see
     * below).
     */
     
    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
    
    
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) { //当工作线程数小于corePoolSize时，使用threadFactory新建线程
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) { //当此时工作线程数已经达到corePoolSize，那么将新增任务加入到workQueue队列
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false)) //此时加入到队列失败即workQueue.offer返回false，也就是队列满了，则开启新新线程进行任务执行
            reject(command); //如果新增后的总工作线程数大于maxPoolSize，那么此时将执行任务拒绝操作，也就是执行RejectedExecutionHandler.rejectedExecution方法。
    }
    
    /**
     * firstTask不为空，表明是execute方法新加进来的worker任务
    */
    private boolean addWorker(Runnable firstTask, boolean core) {
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN &&  //大于SHUTDOWN也就是正数的状态，如果是STOP，TIDYING和TERMINATED，则直接返回false退出。
                ! (rs == SHUTDOWN && //如果是SHUTDOWN，不添加firstTask（因为null），workQueue不为空，也就是此时在消费已经提交的到workQueue的任务，并处理这些任务。
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                return false;

            for (;;) {
                int wc = workerCountOf(c);
                //因为可能并发多个线程进入此处，当此时workerCount不合法，那么返回错误，要么是队列还未满，不能加入到工作线程池，要么是队列满了，工作线程数也超过了maxPoolSize
                if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize))
                    return false;
                if (compareAndIncrementWorkerCount(c)) //采用CAS原子递增workerCount
                    break retry;
                c = ctl.get();  // Re-read ctl
                if (runStateOf(c) != rs)
                    continue retry;
                // else CAS failed due to workerCount change; retry inner loop
            }
        }

        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                  chi  int rs = runStateOf(ctl.get());

                    if (rs < SHUTDOWN ||  //再次检查线程池状态，不允许是STOP，TIDYING和TERMINATED。如果是SHUTDOWN也不允许，再次添加新任务，只允许执行老任务，也就是firstTask == null
                        (rs == SHUTDOWN && firstTask == null)) {
                        if (t.isAlive()) // precheck that t is startable 如果从threadFactory生产出来的线程已经被start，那么不能再此处被二次start，认为是非法状态。
                            throw new IllegalThreadStateException();
                        workers.add(w); //实际工作的线程集合
                        int s = workers.size();
                        if (s > largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)  //假如出现异常，也就是线程未被启动，那么从workers删除该worker。
                addWorkerFailed(w);
        }
        return workerStarted;
    }
    
    final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            //
            while (task != null || (task = getTask()) != null) {
                w.lock(); //加锁，为了completedTasks递增计数，shutdown下进程进行不被interrupt
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted()) //防止虽然STOP，但是还没有执行到interruptWorkers，当然interrupt两次和一次效果是幂等的。
                    wt.interrupt();
                try {
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        task.run(); //这里执行实际实现Runnable接口的类方法
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally { //此时说明上述线程执行过程中出错，尝试清除多余线程，如果不够则新增线程。
            processWorkerExit(w, completedAbruptly);
        }
    }
    
    private Runnable getTask() {
        boolean timedOut = false; // Did the last poll() time out?

        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // 如果是SHUTDOWN并且workQueue为空，或者直接就是STOP，TIDYING，TERMINATED，那么直接退出，并且将会销毁该线程
            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                decrementWorkerCount();
                return null;
            }

            int wc = workerCountOf(c);

            // Are workers subject to culling?
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

            if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
                if (compareAndDecrementWorkerCount(c)) //当前线程需要被清除
                    return null;
                continue;
            }

            try {
                Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) : //超过一定时间，如果未取到数据，则返回null，也就是丢弃该线程
                    workQueue.take();//无限等待，直到从队列取到数据，也就是常驻该线程。假如此时shutdown或shutdownNow，将被唤醒并被catch到异常。
                if (r != null) 
                    return r;
                timedOut = true; //这个时候超时了，还没获取到任务Runnable对象，回到上述代码进行线程清除操作
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }
    
    /**
     * 该方法处理，正常线程执行失败的场景，或者线程超过corePoolSize超时或达到允许corePoolSize线程清除的条件也即allowCoreThreadTimeOut=true等条件。
     * 如果线程数超过某个条件值，则直接return丢弃当前线程，如果是小于，那么则新建一个线程。
    */
    private void processWorkerExit(Worker w, boolean completedAbruptly) {
        if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
            decrementWorkerCount();

        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            completedTaskCount += w.completedTasks;
            workers.remove(w); //从workers中移除当前线程任务
        } finally {
            mainLock.unlock();
        }

        tryTerminate();

        int c = ctl.get();
        if (runStateLessThan(c, STOP)) {
            if (!completedAbruptly) {
                int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
                if (min == 0 && ! workQueue.isEmpty())
                    min = 1;
                if (workerCountOf(c) >= min) //如果线程数大于所需数量，此时直接退出
                    return; // replacement not needed
            }
            //有机会比如不大corePoolSize，就新建线程，此时参与Runnable参数为null，意思是处理queue中的任务
            addWorker(null, false);
        }
    }
    
    private void addWorkerFailed(Worker w) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (w != null)
                workers.remove(w);
            decrementWorkerCount();
            tryTerminate();
        } finally {
            mainLock.unlock();
        }
    }
    
    final void tryTerminate() {
        for (;;) {
            int c = ctl.get();
            if (isRunning(c) || //如果是running状态
                runStateAtLeast(c, TIDYING) || //如果是TIDYING或TERMINATED状态
                (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty())) //如果是SHUTDOWN状态并且是workQueue不会为空
                return;
            //进入此处，此时状态是STOP或者是SHUTDOWN但是workQueue为空
            if (workerCountOf(c) != 0) { // Eligible to terminate 如果当前工作线程不为空
                interruptIdleWorkers(ONLY_ONE);
                return;
            }

            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {// 只有一个线程可以设置成功，CAS实际上已经满足并发控制，这里做个lock，是为了signalAll。
                if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                    try {
                        terminated();
                    } finally {
                        ctl.set(ctlOf(TERMINATED, 0));
                        termination.signalAll(); // 唤醒awaitTermination
                    }
                    return;
                }
            } finally {
                mainLock.unlock();
            }
            // else retry on failed CAS
        }
    }
    
    public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess(); //检查是否有权限关闭
            advanceRunState(SHUTDOWN); //设置状态
            interruptIdleWorkers(); //对所有空闲的worker发起interrupt，针对queue为空的等待时的唤醒使线程退出
            onShutdown(); // hook for ScheduledThreadPoolExecutor
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
    }
    
    private void advanceRunState(int targetState) {
        for (;;) {
            int c = ctl.get(); 
            //只允许设置更高的状态
            if (runStateAtLeast(c, targetState) ||
                // 重新结算新的cdtl变量，然后尝试CAS原子性设置
                ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))
                break;
        }
    }
    
    private void interruptIdleWorkers(boolean onlyOne) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            for (Worker w : workers) {
                Thread t = w.thread;
                if (!t.isInterrupted() && w.tryLock()) {
                    try {
                        t.interrupt();//只interrupt空闲的worker，主要作用在shutdown并且queue队列为空时，使线程worker线程退出。
                    } catch (SecurityException ignore) {
                    } finally {
                        w.unlock();
                    }
                }
                if (onlyOne) // 只需一次interrupt可以让还在blockingQueue等待的线程唤醒一个个被唤醒，因为interruptIdleWorkers(false)可以
                //一次性将大部分block住的线程interrupt，但是一些已经获取lock的线程，可能会再次getTask()而被阻塞住，比如queue突然加了个新task，
                // 但是当前线程池数比queue里的数量还多，那么必然有一些线程会被block住，比如有个线程获取task，执行必然会发现queuew为空退出，触发tryTerminate，
                // 故这里只需一次interrupt，然后一个个扩散开。
                    break;
            }
        } finally {
            mainLock.unlock();
        }
    }
    
    public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            advanceRunState(STOP);
            interruptWorkers();
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
        return tasks;
    }
    private void interruptWorkers() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            for (Worker w : workers)
                w.interruptIfStarted();
        } finally {
            mainLock.unlock();
        }
    }
    
    private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
    {
        /**
         * This class will never be serialized, but we provide a
         * serialVersionUID to suppress a javac warning.
         */
        private static final long serialVersionUID = 6138294804551838833L;

        /** Thread this worker is running in.  Null if factory fails. */
        final Thread thread;
        /** Initial task to run.  Possibly null. */
        Runnable firstTask;
        /** Per-thread task counter */
        volatile long completedTasks;

        /**
         * Creates with given first task and thread from ThreadFactory.
         * @param firstTask the first task (null if none)
         */
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }

        /** Delegates main run loop to outer runWorker  */
        public void run() {
            runWorker(this);
        }

        // Lock methods
        //
        // The value 0 represents the unlocked state.
        // The value 1 represents the locked state.

        protected boolean isHeldExclusively() {
            return getState() != 0;
        }

        protected boolean tryAcquire(int unused) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        protected boolean tryRelease(int unused) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        // 控制completedTasks原子性递增
        public void lock()        { acquire(1); }
        // interruptIdleWorkers中确保不interrupt正在运行的worker
        public boolean tryLock()  { return tryAcquire(1); }
        public void unlock()      { release(1); }
        public boolean isLocked() { return isHeldExclusively(); }

        void interruptIfStarted() {
            Thread t;
            if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                }
            }
        }
    }
    
    
```


## AbstractExecutorService
实现基本的提交任务方法，比如submit(Runnable task),  submit(Callable<T> task)。实际上所有的提交的任务都被封装到RunnableFuture对象，然后再执行线程任务。

## 优先级线程池
由于PriorityBlockingQueue线程需要里面元素进行比较需要元素实现Comparable接口，故ThreadPoolExecutor线程池中的workQueue添加对象既实现Runnable接口的对象也要求实现Comparable接口。