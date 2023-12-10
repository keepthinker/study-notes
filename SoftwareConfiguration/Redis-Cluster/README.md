## 配置Redis Cluster集群

### 示例redis.conf配置文件

```properties
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes

# 自测用配置，允许其他机器不用密码也可以登录
bind 0.0.0.0
protected-mode no


```

### 创建各主从节点目录

```shell
mkdir cluster-test
cd cluster-test
mkdir 7000 7001 7002 7003 7004 7005
```

### 将redis.conf复制到各个目录，并修改port为各目录数字对应的值

```shell
cd 7000
redis-server ./redis.conf
```

### 初始化集群

--cluster-replicas表明一个主节点的副本数量

```shell
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
--cluster-replicas 1
```

### 集群创建成功后显示

```shell
[OK] All 16384 slots covered
```

## 集群一些命令

```shell
> cluster nodes
> fb799eb1a9d34349b89af14a2c1ccf5fd4a1652b 127.0.0.1:7002@17002 myself,master - 0 1652712705000 3 connected 10923-16383
91f2a14113312aee23a5c6dbaa319d829c5aeb6c 127.0.0.1:7000@17000 master - 0 1652712705807 1 connected 0-5460
8798fc3639a888b7598ed1b662b78a46e4a51c7e 127.0.0.1:7004@17004 slave 91f2a14113312aee23a5c6dbaa319d829c5aeb6c 0 1652712704000 1 connected
0a3a19127223cd2d8b3d23e55dce4535d1555e71 127.0.0.1:7005@17005 slave b5f0ebb7dd4318f94de788d601cf20237bd024e4 0 1652712705503 2 connected
8f18f4ac6d141e56047692b0fbb17d31323497e7 127.0.0.1:7003@17003 slave fb799eb1a9d34349b89af14a2c1ccf5fd4a1652b 0 1652712704799 3 connected
b5f0ebb7dd4318f94de788d601cf20237bd024e4 127.0.0.1:7001@17001 master - 0 1652712705502 2 connected 5461-10922

> cluster info
> cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:3
cluster_stats_messages_ping_sent:710
cluster_stats_messages_pong_sent:721
cluster_stats_messages_meet_sent:1
cluster_stats_messages_sent:1432
cluster_stats_messages_ping_received:721
cluster_stats_messages_pong_received:711
cluster_stats_messages_received:1432
total_cluster_links_buffer_limit_exceeded:0
```

## 参考

[Scaling with Redis Cluster | Redis](https://redis.io/docs/manual/scaling/#creating-and-using-a-redis-cluster)
[Docker 搭建 Redis Cluster 集群环境](https://xie.infoq.cn/article/a5536b928edd12beb32fcabf9)
