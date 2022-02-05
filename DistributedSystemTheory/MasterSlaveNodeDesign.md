# 利用现有高可用系统进行master选取
## zookeeper
### 排他锁
排他锁（Exclusive Locks），又被称为写锁或独占锁，如果事务T1对数据对象O1加上排他锁，那么整个加锁期间，只允许事务T1对O1进行读取和更新操作，其他任何事务都不能进行读或写。

定义锁：

/exclusive_lock/lock

实现方式：

利用 zookeeper 的同级节点的唯一性特性，在需要获取排他锁时，所有的客户端试图通过调用 create() 接口，在 /exclusive_lock 节点下创建临时子节点 /exclusive_lock/lock，最终只有一个客户端能创建成功，那么此客户端就获得了分布式锁。同时，所有没有获取到锁的客户端可以在 /exclusive_lock 节点上注册一个子节点变更的 watcher 监听事件，以便重新争取获得锁。
```shell
create /exclusive_lock

# 只有一个客户端获取锁
create -e /exclusive_lock/lock

# 没有获取锁的客户端监听锁是否变更，也就是监听/exclusive_lock的子节点是否变更，如果变更则立马尝试create一个锁。
ls -w /exclusive_lock
```

### 共享锁
共享锁（Shared Locks），又称读锁。如果事务T1对数据对象O1加上了共享锁，那么当前事务只能对O1进行读取操作，其他事务也只能对这个数据对象加共享锁，直到该数据对象上的所有共享锁都释放。

定义锁:

/shared_lock/[hostname]-请求类型W/R-序号

实现方式：

1. 客户端调用 create 方法创建类似定义锁方式的临时顺序节点。
2. 客户端调用 getChildren 接口来获取所有已创建的子节点列表。
3. 判断是否获得锁，对于读请求如果所有比自己小的子节点都是读请求或者没有比自己序号小的子节点，表明已经成功获取共享锁，同时开始执行度逻辑。对于写请求，如果自己不是序号最小的子节点，那么就进入等待。
4. 如果没有获取到共享锁，读请求向比自己序号小的最后一个写请求节点注册 watcher 监听，写请求向比自己序号小的最后一个节点注册watcher 监听。

```shell
create /shared_lock
# 成功创建读锁
create -e -s /shared_lock/192.168.0.1-r-
# 比该节点序列号小的节点都是读锁，那么表示当前客户端获取读锁
create -e -s /shared_lock/192.168.0.1-r-
# 比该节点序列号小的节点存在，那么此时客户端进入等待状态，并通过get -w机制监听前一个节点事件
create -e -s /shared_lock/192.168.0.1-w-
get -w /shared_lock/192.168.0.1-r-0000000001

```


### 参考文献
[掌握 zookeeper 命令，这篇文章就够了](https://blog.csdn.net/feixiang2039/article/details/79810102)

## redis主从
## TiDB利用sql原子性原理进行选取

# 本身集群自选master
## raft协议的master选取方法
