# 角色
1. **消息系统**。和传统消息系统都具备解耦、冗余存储、流量削峰、缓冲、异步通信、扩展性、可恢复性等功能。而且还提供消息顺序性保证和回溯消费的功能。
2. **存储系统**。Kafka把消息持久化到磁盘，相比于其他基于内存系统，有效降低数据丢失到风险。可以把kafka当做传统存储系统来用，只需把数据保留策略设置为“永久”或者开启日志压缩
3. **流式处理平台**。为每个流行的流式处理框架提供了可靠的数据来源，还提供了一个完整的流式处理类库，比如窗口、连接、变换好聚合等各类操作。

# Kafka 架构
- BrokerKafka 集群包含一个或多个服务器，这种服务器被称为 broker
- Topic每条发布到 Kafka 集群的消息都有一个类别，这个类别被称为 Topic。（物理上不同 Topic 的消息分开存储，逻辑上一个 Topic 的消息虽然保存于一个或多个 broker 上但用户需指定消息的 Topic 即可生产或消费数据而不必关心数据存于何处）
- PartitionParition 是物理上的概念，每个 Topic 包含一个或多个 Partition.
- Producer负责发布消息到 Kafka broker
- Consumer消息消费者，向 Kafka broker 读取消息的客户端。
- Consumer Group每个 Consumer 属于一个特定的 Consumer Group（可为每个 Consumer 指定 group name，若不指定 group name 则属于默认的 group）。

# Topic & Partition
Topic 在逻辑上可以被认为是一个 queue，每条消费都必须指定它的 Topic，可以简单理解为必须指明把这条消息放进哪个 queue 里。为了使得 Kafka 的吞吐率可以线性提高，物理上把 Topic 分成一个或多个 Partition，每个 Partition 在物理上对应一个文件夹，该文件夹下存储这个 Partition 的所有消息和索引文件。


# Producer 
KafkaProducer类是线程安全的，可以在多个线程中共享单个KafkaProducer实力。

设定client.id来制定对应的客户端ID。
## 发送消息模式
三种模式：发后既忘、同步及异步。
1. **发后既忘**：只管发，不关心消息是否正确到达。大部分情况没问题，在某些时候会造成消息的丢失。这种发送方式性能最高，可靠性最差。例子：直接调用send方法。
2. **同步**: 
同步发送性能差，需要等待消息发送完之后才能发送下一条。实现同步的方式为利用send方法返回的Future对象实现。例子：producer.send(record).get()
3. **异步**:
一般是在send()方法里制定回个Callback回调函数，Kafka在返回相应时调用该函数来实现异步的发送确认。

## 消息路由

Producer 发送消息到 broker 时，会根据 Paritition 机制选择将其存储到哪一个 Partition。如果 Partition 机制设置合理，所有消息可以均匀分布到不同的 Partition 里，这样就实现了负载均衡。如果一个 Topic 对应一个文件，那这个文件所在的机器 I/O 将会成为这个 Topic 的性能瓶颈，而有了 Partition 后，不同的消息可以并行写入不同 broker 的不同 Partition 里，极大的提高了吞吐率。

在发送一条消息时，可以指定这条消息的 key，Producer 根据这个 key 和 Partition 机制来判断应该将这条消息发送到哪个 Parition。

## 消息流动

消息在通过send()方法发往broker的过程中，有可能需要**依次经过拦截器(Interceptor)，序列化器(Serializer)、分区器(Partitioner)、消息累加器(RecordAccumulator)等**的一系列作用滞后才能被真正地发往broker。

### 拦截器
生产者可以在消息发送前做一些准备工作，比如按照某个规则过滤不符合条件的消息、修改消息的内容。

### 序列化器
生产者需要用序列化器把对象转换成字节数组才能通过网络发送给Kafka。

### 分区器
在默认分区器DefaultPartitioner中，如果key不为null，那么默认的分区器会对key进行哈希，最终根据得到的哈希值来计算分区号，拥有相同的key的消息会被写入同一个分区。如果key为null，则以轮训的方式发往主题内的各个可用分区。

### 消息累加器
消息累加器主要用来缓存消息一边Sender线程可以批量发送，进而减少网络传输的资源以提升性能。缓存大小由参数buffer.memory配置。如果生产者发送消息的速度超过发送到服务器的速度，那么send()方法调用要么阻塞，要么跑出异常，这个取决于max.block.ms的配置。

RecordAccumulator为每个分区都维护一个双端队列，队列元素是ProducerBatch，每个ProducerBatch包含多个ProducerRecord。将多个较小的ProducerRecord包含在ProducerBatch中，可以减少网络请求提高吞吐量。

### InFlightRequests
类似于TCP协议中的滑动窗口。主要是缓存已经发送出去但还没有收到相应的请求。参数配置为max.in.flight.requests.per.connection，默认值为5，超过该数值，就不能再向这个连接发送更多的请求。

## 重要生产者参数

1. acks：这个参数用来指定分区中必须有多少个副本收到这条消息，之后生产者才会认为这条消息是成功写入的。

   ack=1：默认值即为1。此时只要Leader副本成功写入消息，那么他就会收到来自服务器的成功相应。

   ack=0：生产者发送消息之后不需要等待任何服务器的响应。

   ack=-1或ack=all： 生产者在消息发送之后，需要等待ISR中的所有副本都成功写入消息之后才能收到来自服务器的请求。

2. retries和retry.backoff.ms：retries配置重试次数，默认值为0，既异常时不做任何重试。retry.backoff.ms默认值时为100用来设定重试之间的时间间隔。

3. linger.ms：指定生产者发送ProducerBatch之前等待更多消息(ProducerRecord)加入ProducerBatch的时间，默认值为0。

4. request.timeout.ms：等待请求响应的最长时间，默认值为30000ms。


# Consumer

## Consumer Group
使用 Consumer high level API 时，同一 Topic 的一条消息只能被同一个 Consumer Group(消费组)内的一个 Consumer 消费，但多个 Consumer Group 可同时消费这一消息。
![Consumer Group](https://static001.infoq.cn/resource/image/68/22/68f2ca117290d8f438610923c108ce22.png)

实际上，Kafkak的设计理念之一就是同时提供离线处理和实时处理。根据这一特性，可以使用 Storm 这种实时流处理系统对消息进行实时在线处理，同时使用 Hadoop 这种批处理系统进行离线处理，还可以同时将数据实时备份到另一个数据中心，只需要保证这三个操作所使用的 Consumer 属于不同的 Consumer Group 即可。

消费者与消费组这种模型让整体的消费能力具备横向伸缩性，我们可以增加（或减少）消费者的个数来提高（或降低）整体的消费能力。如果消费者的数量大于分区的数量，那么会有消费者分配不到任何分区。kafka通过该模型可以支持点对点模式和发布/订阅模式。消费者根据配置的group.id来指定其所属的消费组。

Kafka API提供灵活的订阅和取消订阅的操作，比如可以订阅正则表达式匹配的topic。

## 消息消费

Kafka消费是基于拉模式，是一个不断轮询的过程，消费者所要做的就是重复地调用poll()方法，而poll()方法返回的是所订阅的主题（分区）上的一组消息。poll()方法里有个timeout来控制阻塞时间，如果消费者缓冲区没有数据时会发生阻塞。

## 位移提交

对于消费者而言，它也有一个offset的概念，消费者使用offset来表示消费到分区中某个消息所在的位置。x表示某一次拉取操作中此分区消息的最大偏移量，但是当前消费者需要提交的消费位移为x+1。

Kafka中默认的消费位移的提交方式为自动提交，这个有消费者客户端参数enable.auto.commit配置，默认值为true。自动提交时定期提交，周期时间由客户端参数auto.commit.interval.ms配置，默认值为5秒，生效前提是enable.auto.commit配置为true。自动提交会带来重复消费的问题。如果需要手动提交位移，那么需要把enable.auto-commit设置为false。手动提交细分为同步提交和异步提交，分别为commitSync()和commitAsync()，可以指定offset方式进行提交，比如使用commitSync(final Map<TopicPartition, OffsetAndMetadata> offsets)方法。

## 控制和关闭消费

Kafka支持使用pause和resume()方法来分别实现暂停从某些分区拉取数据和恢复某些从分区拉取数据操作。使用close()来关闭消费，释放系统资源。

## 指定位移消费

在Kafka中每当消费者查找不到所记录的消费位移时，就会根据消费者客户端参数auto.offset.reset的配置来决定从何处开始消费，默认值为latest，表明从分区末尾开始消费。如果将auto.offset.reset配置为earliest，那么消费者将从起始处，也就是0开始消费。比如当一个新的消费组来消费时，客户端会爆出重置位移的提示信息。

KafkaConsumer中的seek()方法正好提供了这个功能，让我们得以追前消费或回溯消费。通过seek()方法可以让客户端KafkaConsumer指定从beginningOffsets()处（既位置为0处）或从endOffsets处（既指定分区的末尾处）开始消费。

## 再均衡

再均衡是指分区的所属权从一个消费者转移到另一个消费者的行为，它为消费者具备高可用和伸缩性提供保障，使我们可以既方便又安全地删除消费组内的消费者或往消费组里添加消费者。不过在再均衡发生期间，消费组内的消息者是无法消费消息的。如果消费者当前分区被转移，那么消费者将会丢失该分区状态，如果此时消费者消费完一部分消息而没有提交，将会导致另外一个被分配分区的消费者重复消费原来那一部分被消费的消息。

再均衡监听器ComsumerRebalanceListener可以在再均衡操作前后的做一些准备和收尾工作。

## 消费者拦截器

KafkaConsumer会在poll()方法之前调用拦截器的onComsume()方法来对消息进行相应的定制化操作，比如修改返回的消息内容、按照某种规则过滤消息。KafkaConsumer会在提交完消费位移之后调用拦截器的onCommit()方法。

## 多线程实现

KafkaProducer是线程安全的，但是KafkaComsumer是非线程安全的。多线程执行消费有多种方式，其中一种也是最常见的方式：线程封闭，既为每个线程实例化一个KafkaComsumer对象。一个消费线程可以消费一个或多个分区中的消息，所有的消费线程都隶属于同一个消费组。这种方式并发度受限于分区的实际个数，比如当消费线程的个数大于分区数时，就有部分消费线程一直处于空闲的状态。一般情况下一个poll()拉取消息的性能大于之后的处理消息的性能，所以可以在处理消息这里增加一个线程池进行多线程处理，此时需要注意位移提交需要进行加锁处理，防止出现并发问题。



# 主题与分区

主题与分区是提供给上层用户的抽象，而在副本层面或更加确切地说是Log层面才有实际物理上的存在。同一个分区的副本需要分布到不同的broker，这样才能提供有效的冗余。每个副本对应于系统的Log日志文件。Kafka从0.10.x版本开始支持指定broker的机架信息(机架的名称)。如果指定了机架信息，则在分区副本分配时尽可能将分区副本分布到不同的机架上。

主题创建之后，依然允许对其做一定的修改，比如增加分区的个数、修改配置等。目前kafka只支持增加分区数而不支持减少分区数。



## 分区重分配

当集群中一个节点下线，如果节点上的分区是单副本，那么这个分区将不可用，如果是多副本，那么这个节点的Leader副本会转交给其他Follower中。总而言之，如果出现节点不可用，将会影响整个集群的负载均衡，甚至于会影响整体服务的可用性和可靠性。

当集群中新增节点时，旧主题是不会自动分配新节点给自己的分区，只有新主题才会被分配到新节点。

kafka提供kafka-reassign-partitions.sh脚本来执行分区重分配的工作。



# 日志存储

不考虑多副本的情况，一个分区对应一个日志。为了防止日志过大，kafka又引入了日志分段（LogSegment）的概念，将大日志切割成多个LogSegment，便于消息的维护与清理。事实上，Log和LogSegment不是物理意义上的概念，Log在操作系统上以文件夹的形式存储，而每个LogSegment对应于磁盘上的一个日志文件和两个索引文件，以及可能的其他文件（比如以“.txnindex”为后缀的事务索引文件）。

向Log中最佳消息时是顺序写入的，只有最后一个LogSegment才能执行写入操作，在此之前所有的LogSegment都不能写入数据。

## 磁盘存储

Kafka在设计时采用了文件追加的方式来写入消息，既只能在尾部追加消息，而不能修改消息，属于典型的顺序写盘操作。

Kafka大量使用文件系统的页缓存而不是使用Java中的对象来提高吞吐量。虽然消息都是先写入页缓存，然后交由操作系统负责具体的刷盘任务，但在Kafka中同样提供了同步刷盘及间断性强制刷盘功能。但不建议同步刷盘，因为消息的可靠性应该用多副本机制来保证，而不是用同步刷盘这种影响性能的性能来保证。

## 高性能原因

### 顺序写入

磁盘顺序读写速度接近内存随机读写速度。

### Partition设计

为了做到水平扩展，一个topic实际是由多个partition组成的，遇到瓶颈时，可以通过增加partition的数量并将partition分布到不同的物理机，来进行横向扩容。单个parition内是保证消息有序。

### Memory Mapped Files

通过mmap，进程像读写硬盘一样读写内存（当然是虚拟机内存），也不必关心内存的大小有虚拟内存为我们兜底。

使用这种方式可以获取很大的I/O提升，省去了用户空间到内核空间复制的开销（调用文件的read会把数据先放到内核空间的内存中，然后再复制到用户空间的内存中。）

但也有一个很明显的缺陷既不可靠，写到mmap中的数据并没有被真正的写到硬盘，操作系统会在程序主动调用flush的时候才把数据真正的写到硬盘。

Kafka提供了一个参数producer.type来控制是不是主动flush，如果Kafka写入到mmap之后就立即flush然后再返回Producer叫 同步 (sync)；写入mmap之后立即返回Producer不调用flush叫异步 (async)。

## Zero Copy

基于sendfile实现，sendfile系统调用则提供了一种减少以上多次copy，提升文件传输性能的方法。

在内核版本2.1中，引入了sendfile系统调用，以简化网络上和两个本地文件之间的数据传输。sendfile的引入不仅减少了数据复制，还减少了上下文切换。

sendfile(socket, file, len);

运行流程如下：

sendfile系统调用，文件数据被copy至内核缓冲区

再从内核缓冲区copy至内核中socket相关的缓冲区

最后再socket相关的缓冲区copy到协议引擎

相较传统read/write方式，2.1版本内核引进的sendfile已经减少了内核缓冲区到user缓冲区，再由user缓冲区到socket相关缓冲区的文件copy，而在内核版本2.4之后，文件描述符结果被改变，sendfile实现了更简单的方式，系统调用方式仍然一样，细节与2.1版本的 不同之处在于，当文件数据被复制到内核缓冲区时，不再将所有数据copy到socket相关的缓冲区，而是仅仅将记录数据位置和长度相关的数据保存到 socket相关的缓存，而实际数据将由DMA模块直接发送到协议引擎，再次减少了一次copy操作。

### 批量压缩

Kafka使用了批量压缩，即将多个消息一起压缩而不是单个消息压缩。Kafka允许使用递归的消息集合，批量的消息可以通过压缩的形式传输并且在日志中也可以保持压缩格式，直到被消费者解压缩



# Push vs. Pull
作为一个消息系统，Kafka 遵循了传统的方式，选择由 Producer 向 broker push 消息并由 Consumer 从 broker pull 消息。

push 模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的。push 模式的目标是尽可能以最快速度传递消息，但是这样很容易造成 Consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而 pull 模式则可以根据 Consumer 的消费能力以适当的速率消费消息。

对于 Kafka 而言，pull 模式更合适。pull 模式可简化 broker 的设计，Consumer 可自主控制消费消息的速率，同时 Consumer 可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义。

# Kafka delivery guarantee

有这么几种可能的 delivery guarantee：

At most once 消息可能会丢，但绝不会重复传输

At least one 消息绝不会丢，但可能会重复传输

Exactly once 每条消息肯定会被传输一次且仅传输一次，很多时候这是用户所想要的。

总之，Kafka 默认保证 At least once，并且允许通过设置 Producer 异步提交来实现 At most once。而 Exactly once 要求与外部存储系统协作，幸运的是 Kafka 提供的 offset 可以非常直接非常容易得使用这种方式。

# kafka core API
- The Producer API allows an application to publish a stream of records to one or more Kafka topics.
- The Consumer API allows an application to subscribe to one or more topics and process the stream of records produced to them.
- The Streams API allows an application to act as a stream processor, consuming an input stream from one or more topics and producing an output stream to one or more output topics, effectively transforming the input streams to output streams.
- The Connector API allows building and running reusable producers or consumers that connect Kafka topics to existing applications or data systems. For example, a connector to a relational database might capture every change to a table.

# 存储特点
Each partition is an ordered, immutable sequence of records that is **continually appended to—a structured commit log**. The records in the partitions are each assigned a sequential id number called the offset that uniquely identifies each record within the partition.

The Kafka cluster durably **persists all published records**—whether or not they have been consumed—**using a configurable retention period**. Kafka's performance is effectively constant with respect to data size so storing data for a long time is not a problem.

# 消费特点
In fact, the only metadata retained on a per-consumer basis is the offset or position of that consumer in the log. **This offset is controlled by the consumer**: normally a consumer will advance its offset linearly as it reads records, but, in fact, since the position is controlled by the consumer it can consume records in any order it likes.

## 负载均衡与顺序性
By having a notion of parallelism—the partition—within the topics, Kafka is able to **provide both ordering guarantees and load balancing** over a pool of consumer processes. This is achieved by assigning the partitions in the topic to the consumers in the consumer group so that each partition is consumed by exactly one consumer in the group.  By doing this we ensure that the consumer is the only reader of that partition and consumes the data in order.  Note however that there cannot be more consumer instances in a consumer group than partitions.

kafka保证的是分区有序而不是主题有序。

# Stream Processing
It isn't enough to just read, write, and store streams of data, the purpose is to enable real-time processing of streams.

In Kafka a stream processor is anything that takes continual streams of data from input topics, performs some processing on this input, and produces continual streams of data to output topics.

# Replication

The unit of replication is the topic partition. Under non-failure conditions, each partition in Kafka has a single leader and zero or more followers. The total number of replicas including the leader constitute the replication factor. **All reads and writes go to the leader of the partition**. Typically, there are many more partitions than brokers and **the leaders are evenly distributed** among brokers. The logs on the followers are identical to the leader's log—all have the same offsets and messages in the same order (though, of course, at any given time the leader may have a few as-yet unreplicated messages at the end of its log).

分区中所有的副本统称为AR(Assigned Replicas)。所有与Leader副本保持一定程度同步的副本(包括Leader副本)叫做ISR(In-Sync Replicas)。与Leader副本滞后过多的副本叫做OSR(Out-of-Sync 副本)。正常情况下，AR=ISR，OSR=∅。

HW(High Watermark)，既高水位，它标识了消息偏移量，消费者只能取得这个offset之前的消息。ISR集合中最小的LEO即为分区的HW。

LEO(Log End Offset)，标识当前日志文件中下一条待写入消息的offset。

从生产者发出的一条消息首先会被写入分区的leader副本，不过还需要等待ISR集合中的所有follower副本都同步完之后才能认为已经提交，之后才会更新分区的HW，进而消费者可以消费到这条消息。

# 可靠性

### 副本数

越多的副本数越能够保证数据的可靠性，不过过多副本将会影响性能，一般副本数配置为3。

### 生产者参数acks

文章前面已经描述。

### 发送消息模式

文章前面已经描述。如果要求提高可靠性，那么生产者可以采用同步或异步的模式。

### min.insync.replicas

为了防止follower同步太慢，导致所有follower被移出ISR集合导致实际上ack=-1等于ack=1，提供该参数作为辅助，这个参数指定了ISR集合中最小的副本数，如果不满足条件就会抛出异常。

### unclean.leader.election.enable

默认值是false，如果设置为true就意味着当leader下线时候，可以从非ISR集合中选取新的leader，这样有可能造成数据丢失。

### 刷盘策略

broker端有两个参数log.flush.interval.messages和log.flush.interval.ms用来调整刷盘的策略，默认是不做控制而交由操作系统本身来进行处理。通过刷盘极其损耗性能，一般可靠性采用多副本的方式来保证。

### 消费端可靠性

enable.auto.commit默认值为true，开启自动位移提交功能，虽然这种方式非常简便，但它会带来重复消费和消息丢失的问题。如果客户端还没有读取消息，还没成功对消息进行处理，那么此时不应该提交位移，如果消息处理失败，但是提交，则将丢失消息。消费端消费了消息，但是来不及执行提交就宕机了，如果机器再次重启，那么会消费重复消息。

### 回溯消费

Kafka提供一个兜底的功能，即回溯消费，通过这个功能可以让我们能够有机会对漏掉的消息相应地进行回补，进一步提高可靠性。




# 参考文献
[Kafka 设计解析（一）：Kafka 背景及架构介绍](https://www.infoq.cn/article/kafka-analysis-part-1)

[Kafka 2.1 Documentation](https://kafka.apache.org/documentation/)

[Kafka凭什么速度那么快？](https://www.cnblogs.com/binyue/p/10308754.html)