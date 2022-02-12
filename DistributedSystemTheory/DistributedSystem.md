# ACID

In computer science, **ACID** (atomicity, consistency, isolation, durability) is a set of properties of database transactions intended to guarantee data validity despite errors, power failures, and other mishaps.

## 原子性

操作在一次执行过程中要么全部成功，要么全部不执行。

## 一致性

事务在执行之前与执行之后，数据必须处于一致状态。

## 隔离性

在并发环境中，并发的事务是相互隔离的，一个事务的执行不能被其他事务干扰。

- ### 读未递交（READ UNCOMMITTED）
  
  一个事务内可以看到另外一个事务内还未提交的更新。

- ### 读已提交（READ COMMITTED）
  
  一个事务内可以看到另外一个事务已经提交的更新。

- ### 可重复读（REPEATABLE READ)
  
  保证在事务处理过程中，多次读取同一个数据时，其值都和事务开始时刻是一致的。

- ### 串行化（SERIALIZABLE）
  
  所有事务都串行执行。

## 持久性

一个事务一旦提交，它对数据库中对应的数据的状态变更就应该是永久性的。

# CAP

在理论计算机科学中，CAP定理（CAP theorem），又被称作布鲁尔定理（Brewer's theorem），它指出对于一个分布式计算系统来说，不可能同时满足以下三点：Consistency, Availability, Partition Tolerance

## 一致性

在分布式环境中，一致性指的是在多个副本之间是否能够保证一致的特性。等同于所有节点访问同一份最新的数据副本。

## 可用性

系统提供的服务必须一直处于可用的状态，对于用户的每个操作请求总是能够在有限的时间内返回结果。每次请求都能获取到非错的响应——但是不保证获取的数据为最新数据。

## 分区容错性

分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务器，除非是整个网络环境都发生故障。以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

## 只能保证其中两点

根据定理，分布式系统只能满足三项中的两项而不可能满足全部三项。

**CA without P**：若是单节点则不存在P问题，当然也就不是分布式系统。而分布式系统是多节点，是不存在没有P的情况。

**CP without A**：如果不要求A（可用），相当于每个请求都需要在Server之间强一致，而P（分区）会导致同步时间无限延长，如此CP也是可以保证的。很多传统的数据库分布式事务都属于这种模式。

**AP wihtout C**：要高可用并允许分区，则需放弃一致性。一旦分区发生，节点之间可能会失去联系，为了高可用，每个节点只能用本地数据提供服务，而这样会导致全局数据的不一致性。现在众多的NoSQL都属于此类。

# BASE

Basically Avalible, Soft state, Evetually Consitency

## 基本可用

分布式系统在出现不可预知故障时，允许损失部分可用性。比如响应时间上的损失，反应时间增加。又如功能上的损失，购物网站太过繁忙，引导用户到一个降级页面。

## 软状态

允许数据存在中间状态，并认为该中间状态不会影响系统的整体可用性，既允许系统在数据副本之间进行数据同步的过程中存在延时。

## 最终一致性

所有数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。

# 分布式问题解决

## 分布式锁

### 数据库

#### 1、基于唯一索引的插入

```sql
CREATE TABLE `database_lock` (
    `id` BIGINT NOT NULL AUTO_INCREMENT,
    `resource` int NOT NULL COMMENT '锁定的资源',
    `description` varchar(1024) NOT NULL DEFAULT "" COMMENT '描述',
    PRIMARY KEY (`id`),
    UNIQUE KEY `uiq_idx_resource` (`resource`) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='数据库分布式锁表';
```

#### 2、基于普通的update

##### 预备

初始准备，为之后的多次加锁解锁做准备。

insert into lock_table(version) value(0)

##### 加锁

只有一个加锁请求会成功设置，比如MySQL会返回执行成功的个数为1，如果不为1那么表明被其他加锁请求获取了锁。

update lock_table set version = version + 1 where version = 0

##### 解锁

update locktable set version = 0

##### 其他方法

如果不依赖update成功的个数返回，以下介绍另一种加分布式锁思路。

先准备一条数据insert into lock_table(lockId, version) value(0, 0)。

1. select lockId, version from lock_table检查version是否等于0，如果是，则尝试加锁，设置lockId为全局唯一ID，可以采用雪花算法思路生产唯一ID，执行第2步骤。如果是不是0，则表明有其他客户端的加锁请求，那么当前客户端加锁失败。

2. 执行update lock_table set lockId = $uniqueId, version = version + 1 where version = 0

3. 执行上述SQL后，执行select lockId, version from lock_table，发现version = 1那么检查此时的lockId是不是当前客户端刚才的生成的lockId，如果是，则表明获取锁成功，接着执行步骤4。如果不是，则表明获取锁失败。

4. 加锁后，执行完对应业务逻辑，最后释放锁，执行update lock_table set lockId = 0, version = 0 where id = 1

### Redis

### zookeeper

分布式限速。

系统横向拓展提升系统性能与容量。

[基于数据库实现的分布式锁_朱小厮的博客-CSDN博客_数据库分布式锁](https://blog.csdn.net/u013256816/article/details/92854794))

# Zookeeper

## 分布式锁实现

1.利用序列号生成方式，如果一台机器发现，自己所持有的节点序列号最小，则认为自己持有锁。释放锁时，删除自己节点同时进行一次广播事件通知其他节点。

2.当然也可以创建一个节点，成功创建则获取锁，失败则等待，获取锁的程序，释放锁时，直接删除自己创建的zk节点，然后广播通知给其他节点作锁操作。

## 序列号生成器

每个zk节点都有个版cversion，每次节点变动都会原子自增1。可以通过CAS机制，进行节点版本判断进行数值叠加。

# 参考文献

[谈谈分布式系统的CAP理论](https://zhuanlan.zhihu.com/p/33999708)
