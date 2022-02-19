# 两阶段提交

在计算机网络以及数据库领域内，二阶段提交（英语：Two-phase Commit）是指，为了使基于分布式系统架构下的所有节点在进行事务提交时保持一致性而设计的一种算法(Algorithm)。

在分布式系统中，每个节点虽然可以知晓**自己的操作时成功或者失败**，却**无法知道其他节点**的操作的成功或失败。当一个**事务**跨越**多个节**点时，为了保持事务的ACID特性，需要引入一个作为协调者的组件来统一掌控所有节点(称作参与者)的操作结果并最终指示这些节点是否要把操作结果进行真正的提交(比如将更新后的数据写入磁盘等等)。

二阶段提交的算法思路可以概括为：参与者将操作成败通知协调者，再由协调者根据所有参与者的反馈情报决定各参与者是否要提交操作还是中止操作。

## 第一阶段(提交请求阶段)

协调者节点向所有参与者节点询问是否可以执行提交操作，并开始等待各参与者节点的响应。
参与者节点执行询问发起为止的所有事务操作，并将Undo信息和Redo信息写入日志。
各参与者节点响应协调者节点发起的询问。如果参与者节点的事务操作实际执行成功，则它返回一个"同意"消息；如果参与者节点的事务操作实际执行失败，则它返回一个"中止"消息。
有时候，第一阶段也被称作投票阶段，即各参与者投票是否要继续接下来的提交操作。

## 第二阶段(提交执行阶段)

### 成功

当协调者节点从所有参与者节点获得的相应消息都为"同意"时：

协调者节点向所有参与者节点发出"正式提交"的请求。
参与者节点正式完成操作，并**释放**在整个事务期间内占用的**资源**。
参与者节点向协调者节点发送"完成"消息。
协调者节点收到所有参与者节点反馈的"完成"消息后，完成事务。

### 失败

如果任一参与者节点在第一阶段返回的响应消息为"终止"，或者 协调者节点在第一阶段的询问超时之前无法获取所有参与者节点的响应消息时：

协调者节点向所有参与者节点发出"回滚操作"的请求。
参与者节点利用之前写入的Undo信息执行回滚，并释放在整个事务期间内占用的资源。
参与者节点向协调者节点发送"回滚完成"消息。
协调者节点收到所有参与者节点反馈的"回滚完成"消息后，取消事务。
有时候，第二阶段也被称作完成阶段，因为无论结果怎样，协调者都必须在此阶段结束当前事务。

## 算法示意

下述流程图简单示意了二阶段提交算法中协调者和参与者之间的通信流程。

```
    协调者                                              参与者
                              QUERY TO COMMIT
                -------------------------------->
                              VOTE YES/NO           prepare*/abort*
                <-------------------------------
commit*/abort*                COMMIT/ROLLBACK
                -------------------------------->
                              ACKNOWLEDGMENT        commit*/abort*
                <--------------------------------  
end


"*" 所标记的操作意味着此类操作必须记录在稳固存储上.[1]
```

## 缺点? -- 与原始定义有异议

1、同步阻塞问题。
执行过程中，**所有参与节点都是事务阻塞型的**。当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态。也就是说从投票阶段到提交阶段完成这段时间，资源是被锁住的。

2、单点故障。由于协调者的重要性，一旦协调者发生故障。参与者会一直阻塞下去。
尤其在第二阶段，协调者发生故障，那么所有的参与者还都处于锁定事务资源的状态中，而无法继续完成事务操作。
【协调者发出Commmit消息之前宕机的情况】
（如果是协调者挂掉，可以重新选举一个协调者，但是无法解决因为协调者宕机导致的参与者处于阻塞状态的问题）

3、数据不一致。在二阶段提交的阶段二中，当协调者向参与者发送commit请求之后，发生了局部网络异常或者在发送commit请求过程中协调者发生了故障，这回导致**只有一部分参与者接受到了commit请求**。而在这部分参与者接到commit请求之后就会执行commit操作。但是其他部分未接到commit请求的机器则无法执行事务提交。于是整个分布式系统便出现了数据不一致性的现象。

4、二阶段无法解决的问题：------ 极限情况下,对某一事务的不确定性！
协调者在发出commit消息之后宕机，而唯一接收到这条消息的参与者同时也宕机了。那么即使协调者通过选举协议产生了新的协调者，这条事务的状态也是不确定的，没人知道事务是否被已经提交。

# 三阶段提交

In computer networking and databases, the three-phase commit protocol (3PC)[1] is a distributed algorithm which lets all nodes in a distributed system agree to commit a transaction. Unlike the two-phase commit protocol (2PC) however, 3PC is non-blocking. Specifically, 3PC **places an upper bound on the amount of time required** before a transaction either commits or aborts. This property ensures that if a given transaction is attempting to commit via 3PC and holds some resource locks, it will release the locks after the **timeout**.

## Protocol Description

![image](https://upload.wikimedia.org/wikipedia/en/3/39/Three-phase_commit_diagram.png)

### Coordinator

1. The coordinator receives a transaction request. If there is a failure at this point, the coordinator aborts the transaction (i.e. upon recovery, it will consider the transaction aborted). Otherwise, the coordinator sends a canCommit? message to the cohorts and moves to the waiting state.

2. If there is **a failure**, **timeout**, or if the coordinator **receives a No message** in the waiting state, the coordinator aborts the transaction and sends an abort message to all cohorts. Otherwise the coordinator will receive Yes messages from all cohorts within the time window, so it sends preCommit messages to all cohorts and moves to the prepared state.

3. If the coordinator succeeds in the prepared state, it will move to the commit state. However if the coordinator **times out while waiting for an acknowledgement from a cohort**, it will abort the transaction. In the case where an acknowledgement is received from the majority of cohorts, the coordinator moves to the commit state as well.

### Cohort

1. The cohort receives a canCommit? message from the coordinator. If the cohort agrees it sends a Yes message to the coordinator and moves to the prepared state. Otherwise it sends a No message and aborts. If **there is a failure**, it moves to the abort state.

2. In the prepared state, if the cohort **receives an abort message** from the coordinator, **fails**, or **times out waiting for a commit**, it aborts. If the cohort receives a preCommit message, it sends an ACK message back and awaits a final commit or abort.

3. If, after a cohort member receives a preCommit message, the coordinator **fails** or **times out**, the cohort member goes forward with the commit.

参与者回复协调者因为网络不通超时，实际上等同于协调者因为网络不通而导致发送请求给参与者超时既参与者**接受协调者请求超时**。

### Disadvantages

The main disadvantage to this algorithm is that it cannot recover in the event the network is **segmented** in any manner. The original 3PC algorithm assumes a fail-stop model, where processes fail by crashing and crashes can be accurately detected, and **does not work with network partitions or asynchronous communication**.

Keidar and Dolev's E3PC[2] algorithm eliminates this disadvantage.

The protocol requires **at least three round** trips to complete, needing a minimum of three round trip times (RTTs). This is potentially **a long latency** to complete each transaction.

### 参考文献

[分布式一致性之两阶段提交协议、三阶提交协议](https://zhuanlan.zhihu.com/p/35616810)

[Three-phase commit protocol](https://en.wikipedia.org/wiki/Three-phase_commit_protocol)

[关于分布式事务、两阶段提交协议、三阶提交协议](http://www.hollischuang.com/archives/681)

# Percolator 的事务模型

Percolator 提供了跨行、跨表的、基于快照隔离的ACID事务。

## Snapshop isolation

Percolator 使用Bigtable的时间戳记维度实现了数据的多版本化。多版本化保证了快照隔离snapshot isolation级别，优点如下：

- 对读操作：使得每个读操作都能够从一个带时间戳的稳定快照获取。
- 对写操作，能很好的应对写写冲突：若事务A和B并发去写一个同一个元素，最多只有一个会提交成功。

### 参考文献

[Google Percolator 的事务模型](https://github.com/ngaut/builddatabase/blob/master/percolator/README.md)

## Timestamp Ordering Concurrency Control

### Basic Timestamp Ordering

Every database object X is tagged with timestamp of the last transaction that successfully did read/write:

1. W-TS(X): Write timestamp on object X.
2. R-TS(X): Read timestamp on object X.

The DBMS check timestamps for every operation. If transaction tries to access an object “from the future”, then the DBMS aborts that transaction and restarts it.

1. Read Operations:
   
   <pre>
   (a) If TS(Ti) < W-TS(X) this violates timestamp order of Ti with regard to the writer of X. Thus
   you abort Ti and restart it with same TS.
   (b) Else:
   i. Allow Ti to read X.
   ii. Update R-TS(X) to max(R-TS(X), TS(Ti)).
   iii. Have to make a local copy of X to ensure repeatable reads for Ti.
   iv. Last step may be skipped in lower isolation levels.
   </pre>
   
   Write Operations:
   
   <pre>
   (a) If TS(Ti) < R-TS(X) or TS(Ti) < W-TS(X), abort and restart Ti.
   (b) Else:
   i. Allow Ti to write X and update W-TS(X) to Ti.
   ii. Also have to make a local copy of X to ensure repeatable reads for Ti.

2. Optimization: Thomas Write Rule
   (a) If TS(Ti) < R-TS(X): Abort and restart Ti
   (b) If TS(Ti) < W-TS(X):
   i. Thomas Write Rule: Ignore the write and allow transaction to continue.
   ii. Note that this violates timestamp order of Ti but this is okay because no other transaction
   will ever read Ti’s write to object X.
   (c) Else: Allow Ti to write X and update W-TS(X)
   </pre>

### 个人解释

1. Thomas Write Rule可以理解为既然一个事务从开始时第一个操作是写入，而后又开始读取写入的值，为什么不一开始不写入，而是利用该值直接做其他一些读取后的操作，一般情况下数据库也会缓存写入的值而不是再次从数据库读取相同的值，然后才考虑之后需不需要将该值写入数据库。一个事务一开始写入，假如其他事务也并发写入该值，实际上此时也未有改变该值的一致性，无非是谁先写入的问题，故该事务可以放弃当前写入操作。

### Implementation Issues - Timestamp Locking

Even though this technique is a non-locking one, in as much as the Object is not locked from concurrent access for the duration of a transaction, the act of recording each timestamp against the Object requires an extremely short duration lock on the Object or its proxy.

# Multiversion concurrency control

### 参考文献

[Lecture 19: Timestamp Ordering](https://15445.courses.cs.cmu.edu/fall2017/notes/19-notes-timestampordering.pdf)

[Timestamp-based Concurrentcy Control](https://en.wikipedia.org/wiki/Timestamp-based_concurrency_control)

[Multiversion concurrency control - Wikipedia](https://en.wikipedia.org/wiki/Multiversion_concurrency_control)

[MVCC 原理](https://zhuanlan.zhihu.com/p/147372839)
