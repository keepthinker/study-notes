# 前言
本文为官方文档和多篇博客文章的内容提取，同时整理自己一些个人思路，内容主要为RabbitMQ整体设计情况以及集群的设计理念讲述。在此感谢各位作者的RabbitMQ知识分享。

# 简介
RabbitMQ是一套开源（MPL）的消息队列服务软件，是由 LShift 提供的一个 Advanced Message Queuing Protocol (AMQP) 的开源实现，由以高性能、健壮以及可伸缩性出名的 Erlang 写成。AMPQ，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。

# 基本概念
![image](https://upload-images.jianshu.io/upload_images/120495-de24b12deeb18350.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/530)

1）Message
消息，它由消息头和消息体两部分组成。消息体是不透明的，但消息头是由一些属性组成的，其中包括：routing-key（路由键）、priority（优先权）、delivery-mode（持久存储）。

2）Publisher
生产者，也是消息的生产者，它是向交换器发布消息的应用程序

3）Exchange
交换器，用来接收生产者传递过来的消息，然后将这些消息路由至服务器中的队列

4）Binding
绑定，用于消息队列与交换器之间的沟通。也是消息路由的规则，相当于一个路由表。

5）Queue
消息队列，用来保存消息直到发送给消费者。一个消息可以进入一个或多个队列，除消费者取走消息，否则它一直在消息队列里。

6）Connection
网络连接，如：一个 TCP 连接

7）Channel
信道，多路复用连接中一个独立的双向数据传输通道。无论是发布消息、订阅队列、接收消息都是通过信道来完成。复用信道是为了降低系统资源的消耗。

8）Consumer
消费者，也就是接收生产者发来的消息的客户端应用。

9）Virtual Host
虚拟主机，交换器、消息队列相关的对象。一个VHOST其实可以看成一个 rabbitmp 服务器，它拥有自己的队列、交换器、绑定与权限机制等。Rabbitmq 默认 vhost 是/。

# Rabbitmq 的工作过程

1）客户端连接到消息队列服务器，开启一个 channel

2）客户端声明一个 exchange、queue，并配置相关属性

3）客户端使用 routing key，在 exchange 与 queue 之间建立好绑定关系

4）客户端传递消息到交换器

5）交换器接收到消息后，根据预定的 KEY 与绑定关系，对消息进行路由至消息队列

![image](https://upload-images.jianshu.io/upload_images/120495-d4fcbca2c39f6419?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

## exchange
AMQP 协议中的核心思想就是生产者和消费者的解耦，生产者从不直接将消息发送给队列。生产者通常不知道是否一个消息会被发送到队列中，只是将消息发送到一个交换机。先由 Exchange 来接收，然后 Exchange 按照特定的策略转发到 Queue 进行存储。Exchange 就类似于一个交换机，将各个消息分发到相应的队列中。

### Fanout Exchange
Fanout Exchange 会忽略 RoutingKey 的设置，直接将 Message 广播到所有绑定的 Queue 中。

### Direct Exchange
Direct Exchange 是 RabbitMQ 默认的 Exchange，完全根据 RoutingKey 来路由消息。设置 Exchange 和 Queue 的 Binding 时需指定 RoutingKey（一般为 Queue Name），发消息时也指定一样的 RoutingKey，消息就会被路由到对应的Queue。

### Topic Exchange
Topic Exchange 和 Direct Exchange 类似，也需要通过 RoutingKey 来路由消息，区别在于Direct Exchange 对 RoutingKey 是精确匹配，而 Topic Exchange 支持模糊匹配。分别支持*和#通配符，*表示匹配一个单词，#则表示匹配没有或者多个单词。

# 消息堆积模式
RabbitMQ是典型的内存式堆积，但这并非绝对，在某些条件触发后会有换页动作来将内存中的消息换页到磁盘（换页动作会影响吞吐），或者直接使用惰性队列来将消息直接持久化至磁盘中。

# 多租户
也可以称为多重租赁技术，是一种软件架构技术，主要用来实现多用户的环境下公用相同的系统或程序组件，并且仍可以确保各用户间数据的隔离性。RabbitMQ 就能够支持多租户技术，每一个租户表示为一个vhost，其本质上是一个独立的小型 RabbitMQ 服务器，又有自己独立的队列、交换器及绑定关系等，并且它拥有自己独立的权限。vhost就像是物理机中的虚拟机一样，它们在各个实例间提供逻辑上的分离，为不同程序安全保密地允许数据，它既能将同一个 RabbitMQ中的众多客户区分开，又可以避免队列和交换器等命名冲突。

# 横向拓展能力的思路
搭建多套集群，通过客户端做分片，将生产方生产的消息平均分散到各个集群来实现RabbitMQ集群的横向拓展能力。

# 集群模式
假设所有集群用户可用，一个客户端会连接任意一个节点做操作。对于所支持消息传输协议，客户端每次只会连接一个节点。

假如节点不可用，客户端需要连接其他节点，恢复拓扑其结构继续执行。大部分库维护一个节点列表当做连接选项。有些场景并不能让客户端通过连接一个不同的节点正确继续执行，比如集群的普通模式中的持久化。

RabbitMQ 集群分为两种普通集群和镜像集群，可以说镜像集群是普通集群的晋升版。

## 普通集群
对于Queue来说，消息实体只存在于其中一个节点，rabbit01、rabbit02两个节点仅有相同的元数据，即队列结构，但队列的元数据仅保存有一份，即创建该队列的rabbitmq节点（A节点），当rabbit01节点宕机，你可以去其rabbit02节点查看，./rabbitmqctl list_queues 发现该队列已经丢失，但声明的exchange还存在。

对于Queue来说，消息实体只存在于其中一个节点，当消息进入rabbit01节点的Queue中后，consumer从rabbit02节点拉取时，RabbitMQ会临时在rabbit01、rabbit02间进行消息传输，把在rabbit01中的消息实体取出并经过在rabbit02发送给consumer，所以consumer应平均连接每一个节点，从中取消息。

![image](http://chyufly.github.io/images/RabbitMQ_cluster_imag.PNG)


### 优缺点及其适用场景
该模式存在一个问题就是当rabbit01节点故障后，rabbit02节点无法取到rabbit01。如果做了队列持久化或消息持久化，那么得等rabbit01节点恢复，然后才可被消费，并且在rabbit01节点恢复之前其它节点不能再创建A节点已经创建过的持久队列；如果没有持久化的话，消息就会失丢。

这种模式更适合非持久化队列，只有该队列是非持久的，客户端才能重新连接到集群里的其他节点，并重新创建队列。假如该队列是持久化的，那么唯一办法是将故障节点恢复起来。

### 其他思考
为什么RabbitMQ不将队列复制到集群里每个节点呢？这与它的集群的设计本意相冲突，集群的设计目的就是增加更多节点时，能线性的增加性能（CPU、内存）和容量（内存、磁盘）。理由如下：

Storage Space: If every cluster node had a full copy of every queue, adding nodes wouldn’t give you more storage capacity. For example, if one node could store 1GB of messages, adding two more nodes would simply give you two more copies of the same 1GB of messages.（存储空间：如果每个集群节点每个队列的一个完整副本，增加节点需要更多的存储容量。例如，如果一个节点可以存储1 gb的消息，添加两个节点需要两份相同的1gb的消息）

Performance: Publishing messages would require replicating those messages to every cluster node. For durable messages that would require triggering disk activity on all nodes for every message. Your network and disk load would increase every time you added a node, keeping the performance of the cluster the same (or possibly worse).（性能：发布消息需要将这些信息复制到每个集群节点。对持久消息，要求为每条消息触发磁盘活动在所有节点上。每次添加一个节点都会带来 网络和磁盘的负载。）
当然RabbitMQ新版本集群也支持队列复制（有个选项可以配置）。比如在有五个节点的集群里，可以指定某个队列的内容在2个节点上进行存储，从而在性能与高可用性之间取得一个平衡（应该就是指镜像模式）

## 镜像集群
在普通集群的基础上，把需要的队列做成镜像队列，消息实体会主动在镜像节点间同步，而不是在客户端取数据时临时拉取，也就是说多少节点消息就会备份多少份。

由于镜像队列之间消息自动同步，且内部有选举master机制，即使master节点宕机也不会影响整个集群的使用，达到去中心化的目的，从而有效的防止消息丢失及服务不可用等问题。

![image](http://chyufly.github.io/images/RabbitMQ_image_queues.PNG)

该模式解决了普通模式的问题，其实质不同之处在于，消息实体会主动在镜像节点间同步，而不是在consumer取数据时临时拉取。

### 优缺点及其适用场景
该模式带来的副作用也很明显，除了降低系统性能外，如果镜像队列数量过多，加之大量的消息进入，集群内部的网络带宽将会被这种同步通讯大大消耗掉。所以在对可靠性要求较高的场合中适用。

所以在对可靠性要求较高的场合中适用，一个队列想做成镜像队列，需要先设置policy，然后客户端创建队列的时候，rabbitmq集群根据“队列名称”自动设置是普通集群模式或镜像队列。具体如下：

### 与普通模式的对比
非镜像队列和镜像队列之间是有区别的，前者缺乏额外的镜像基础设施，没有任何slave，因此会运行得更快。





# 参考文献
[RabbitMQ分布式集群架构和高可用性（HA](http://chyufly.github.io/blog/2016/04/10/rabbitmq-cluster/)）

[RabbitMQ技术详解](https://blog.csdn.net/butcher8/article/details/44274731)

[RabbitMQ集群官方文档](http://www.rabbitmq.com/clustering.html)

[深入理解消息中间件技术之RabbitMQ服务](https://www.jianshu.com/p/c9d1ec74aa87)

[RabbitMQ(维基百科)](https://zh.wikipedia.org/wiki/RabbitMQ)

[理解 RabbitMQ Exchange](https://zhuanlan.zhihu.com/p/37198933)