# RocketMQ

## Domain Model

Apache RocketMQ is a distributed middleware service that adopts an asynchronous communication model and a publish/subscribe message transmission model.

![img](domain-apache-rocketmq.png)

As shown in the preceding figure, the lifecycle of a Apache RocketMQ message consists of three stages: production, storage, and consumption.

A producer generates a message and sends it to a Apache RocketMQ broker. The message is stored in a topic on the broker. A consumer subscribes to the topic to consume the message.

**Message production**

Producer：

The running entity that is used to generate messages in Apache RocketMQ. Producers are the upstream parts of business call links. Producers are lightweight, anonymous, and do not have identities.

**Message storage**

- Topic：
  
  The grouping container that is used for message transmission and storage in Apache RocketMQ. A topic consists of multiple message queues, which are used to store messages and scale out the topic.

- MessageQueue：
  
  The unit container that is used for message transmission and storage in Apache RocketMQ. Message queues are similar to partitions in Kafka. Apache RocketMQ stores messages in a streaming manner based on an infinite queue structure. Messages are stored in order in a queue.

- Message：
  
  The minimum unit of data transmission in Apache RocketMQ. Messages are immutable after they are initialized and stored.

**Message consumption**

- ConsumerGroup：
  
  An independent group of consumption identities defined in the publish/subscribe model of Apache RocketMQ. A consumer group is used to centrally manage consumers that run at the bottom layer. Consumers in the same group must maintain the same consumption logic and configurations with each other, and consume the messages subscribed by the group together to scale out the consumption capacity of the group.

- Consumer：
  
  The running entity that is used to consume messages in Apache RocketMQ. Consumers are the downstream parts of business call links, A consumer must belong to a specific consumer group.

- Subscription：
  
  The collection of configurations in the publish/subscribe model of Apache RocketMQ. The configurations include message filtering, retry, and consumer progress Subscriptions are managed at the consumer group level. You use consumer groups to specify subscriptions to manage how consumers in the group filter messages, retry consumption, and restore a consumer offset.
  
  The configurations in a Apache RocketMQ subscription are all persistent, except for filter expressions. Subscriptions are unchanged regardless of whether the broker restarts or the connection is closed.

### Message transmission model[​](https://rocketmq.apache.org/docs/domainModel/01main#message-transmission-model "Direct link to heading")

Message middleware services have two common transmission models: the point-to-point model and the publish/subscribe model.

Point-to-point model

![img](point-to-point-model.png)

The point-to-point model, also known as the queue model, has the following characteristics:

- Consumer anonymity: The queue is the only identity used during upstream-downstream communication. Downstream consumers cannot declare an identity when they obtain messages from the queue.

- One-to-one communication: Consumers do not have identities. All consumers in a consumer group consume the subscribed messages together. Each message can be consumed only by one specific consumer. For this reason, this model supports only one-to-one communication.

Publish/subscribe model

![img](publish-subscribe-model.png)

This model has the following characteristics:

- Independent consumption: In this model, consumers use the identity of a consumer group, or a subscription, to receive and consume messages. Consumer groups are independent of each other.

- One-to-many communication: Based on the design of independent identity, this model allows a topic to be subscribed to by multiple consumer groups, each having full access to all the messages. Therefore, the publish/subscribe model supports one-to-many communication.

Comparison between transmission models

The point-to-point model is simpler in structure, while the publish/subscribe model offers better scalability. Apache RocketMQ uses and has the same high scalability as the publish/subscribe model.

Comparison between transmission models

The point-to-point model is simpler in structure, while the publish/subscribe model offers better scalability. Apache RocketMQ uses and has the same high scalability as the publish/subscribe model.

## Topic

This section describes the definition, model relationship, internal attributes, and behavior constraints of topics in Apache RocketMQ. This topic also provides version compatibility information and usage notes for topics.

### Definition[​](https://rocketmq.apache.org/docs/domainModel/02topic#definition "Direct link to heading")

A topic is logically a collection of queues; we may publish messages to or receive from it.

Topics provide the following benefits:

- **Message categorization and message isolation**: When you create a messaging service based on Apache RocketMQ, we recommend that you use different topics to manage messages of different business types for isolated storage and subscription.

- **Identity and permission management**: Messages in Apache RocketMQ are anonymous. You can use a topic to perform identity and permission management for messages of a specific category.

In Apache RocketMQ, a topic is a top-level storage container in which all message resources are defined. A topic is a logical concept and not the actual unit that stores messages.

A topic contains one or more queues. Message storage and scalability are implemented based on queues. All constraints and attribute settings for a topic are implemented based on the queues in the topic.

### Internal attributes[​](https://rocketmq.apache.org/docs/domainModel/02topic#internal-attributes "Direct link to heading")

**Topic name**

- Definition: the name of a topic. A topic name identifies the topic and is globally unique in a cluster.

- Value: A topic name is specified by the user when a topic is created.

- Constraint: See [Parameter limits](https://rocketmq.apache.org/docs/introduction/03limits).

**Queues**

- Definition: the actual storage unit that stores messages. A topic contains one or more queues. For more information, see [Message queues](https://rocketmq.apache.org/docs/domainModel/03messagequeue).

- Value: You can specify the number of queues when you create a topic. Apache RocketMQ allocates the specified number of queues to the topic.

- Constraint: A topic must contain at least one queue.

**Message type**

- Definition: the message type that is specified for a topic.

- Value: When you create a topic in Apache RocketMQ, select one of the following message types for the topic:
  
  - Normal: [Normal messages](https://rocketmq.apache.org/docs/featureBehavior/01normalmessage). A normal message does not require special semantics and is not correlated with other normal messages.
  
  - FIFO: [Fifo messages](https://rocketmq.apache.org/docs/featureBehavior/03fifomessage). Apache RocketMQ uses a message group to determine the order of a specified set of messages. The messages are delivered in the order in which they are sent.
  
  - Delay: [Delayed messages](https://rocketmq.apache.org/docs/featureBehavior/02delaymessage). You can specify a delay to make messages available to consumers only after the delay has elapsed, instead of delivering messages immediately when they are produced.
  
  - Transaction: [Transaction messages](https://rocketmq.apache.org/docs/featureBehavior/04transactionmessage). Apache RocketMQ supports distributed transaction messages and ensures transaction consistency of database updates and message calls.

- Constraint: Starting from version 5.0, Apache RocketMQ supports enforcing the validation of message types, that is, each topic only allows messages of a single type to be sent. This can better facilitate operation and management of production systems and avoid confusion. However, to ensure backward compatibility with version 4.x, the validation feature is disabled by default. It is recommended to enable it manually through the server parameter "enableTopicMessageTypeCheck".

### Usage Example

For creating topics in Apache RocketMQ 5.0, it is recommended to use the mqadmin tool. However, it is worth noting that message type needs to be added as a property parameter. The following is an example:

```bash
sh mqadmin updateTopic -n <nameserver_address> -t <topic_name> -c <cluster_name> -a +message.type=<message_type>
```

Among these, the message_type parameter can be set as Normal/FIFO/Delay/Transaction based on the message type. If it is not specified, it defaults to the Normal type.

## Message Queue

### Definition

A queue is a container that is used to store and transmit messages in Apache RocketMQ. A queue is the smallest unit of storage for Apache RocketMQ messages.

A topic in Apache RocketMQ consists of multiple queues. This way, queues support horizontal partitioning and streaming storage.

Queues provide the following benefits:

- Ordered storageQueues are ordered in nature. Messages are stored in the same order in which they are queued. The earliest message is at the start of the queue and the latest message is at the end of the queue. Offsets are used to label the locations and the order of messages in a queue.

- Streaming operation semantics. The queue-based storage in Apache RocketMQ allows consumers to read one or more messages from an offset. This helps implement features such as aggregate read and backtrack read. These features are not available in RabbitMQ or ActiveMQ.

### Model relationship

By default, Apache RocketMQ provides reliable message storage. All messages that are successfully delivered are persistently stored in queues. Messages are sent by the producer and received by the consumer client. Each message can be successfully delivered at least once.

The queue model of Apache RocketMQ is similar to the partition model of Kafka. In Apache RocketMQ, a queue is part of a topic. Messages are operated in queues even though they are managed by topic. For example, when a producer sends a message to a specific topic, the message is sent to a queue in the topic.

## Message

The characteristics of the message model in Apache RocketMQ are:

- **Immutability**: A message is an event that is generated. After the message is generated, the content of the message does not change. Even if the message passes through a transmission channel, the content of the message remains the same. The messages that consumers obtain are read-only messages.

- **Persistence**: By default, Apache RocketMQ persists messages. The received messages are stored in the storage file of the Apache RocketMQ broker to ensure that the messages can be traced and restored if system failures occur.

## 

## Producer

### Definition

A producer is typically integrated on the business system and serves to encapsulate data as messages in Apache RocketMQ and send the messages to the server.

The following message delivery elements are defined on the producer side:

- Transmission mode: A producer can specify the message transmission mode in an API operation. Apache RocketMQ supports synchronous transmission and asynchronous transmission.

- Batch transmission: A producer can specify batch transmission in an API operation. For example, the number or size of messages sent at a time can be specified.

- Transactional behavior: Apache RocketMQ supports transaction messages. Producers are involved in transaction checks to ensure eventual consistency of transactions.

Producers and topics have an n-to-n relationship. A producer can send messages to multiple topics, and a topic can receive messages from multiple producers. This many-to-many relationship facilitates performance scaling and disaster recovery.

## Consumer Group

### Definition

A consumer group is a load balancing group that contains consumers that use the same consumption behaviors in Apache RocketMQ.

Unlike consumers that are running entities, consumer groups are logical resources. Apache RocketMQ initializes multiple consumers in a consumer group to achieve the scaling of consumption performance and high availability disaster recovery.

In a consumer group, consumers consume messages based on the consumption behaviors and load balancing policy that are defined in the group. The following section describes the consumption behaviors that are defined:

- Subscription: Apache RocketMQ manages and traces subscriptions based on consumer groups. 

- Delivery order: The Apache RocketMQ broker delivers messages to consumers by using ordered delivery or concurrent delivery. You can configure the delivery method in the consumer group.

- Consumption retry policy: the retry policy that is used when a consumer fails to consume a message. The policy includes the number of retries and the setting of dead-letter queues.

- 

### Internal attributes[​](https://rocketmq.apache.org/docs/domainModel/07consumergroup#internal-attributes "Direct link to heading")

**Consumer group name**

- Definition: the name of a consumer group. Consumer group names are used to distinguish between consumer groups. Consumer group names are **globally unique** in a cluster.

- Values: created and configured by users.

**Delivery order**

- Definition: the order in which Apache RocketMQ delivers messages to a consumer client.
  
  Apache RocketMQ supports ordered delivery and concurrent delivery based on different consumption scenarios.

- Values: The default delivery method is concurrent delivery.

**Consumption retry policy**

- Definition: the retry policy that is used when a consumer fails to consume a message. If a consumer fails to consume a message, the system re-delivers the failed message to the consumer for re-consumption based on the policy.

- Values:A consumption retry policy contains the following items:
  
  - Maximum retries: the maximum number of times that a message can be re-delivered. If a message fails to be consumed and the maximum number of retries is exceeded, the message is delivered to the dead-letter queue or is discarded.
  
  - Retry interval: the interval between which the Apache RocketMQ broker re-delivers a failed message.

- Constraint: Retry interval is available only for push consumers.

**Subscription**

- Definition: the set of subscription relationships that are associated with the current consumer group. A subscription includes the topics to which the consumers subscribe and the message filter rules that are used by consumers. 

Consumers dynamically register subscriptions for consumer groups. The Apache RocketMQ broker persists subscriptions and matches the subscriptions to the consumption progress of messages.

## Consumer

### Definition

A consumer is an entity that receives and processes messages in Apache RocketMQ.

Consumers are usually integrated in business systems. They obtain messages from Apache RocketMQ brokers and convert the messages into information that can be perceived and processed by business logic.

The following items determine consumer behavior:

- Consumer identity: A consumer must be associated with a consumer group to obtain behavior settings and consumption status.

- Consumer type: Apache RocketMQ provides a variety of consumer types for different development scenarios, including push consumers, simple consumers and pull consumers.

- Local settings for consumers: These settings specify how consumer clients run based on the consumer type. For example, you can configure the number of threads and concurrency settings on consumers to achieve different transmission effects.

## Subscription

### Definition

A subscription is the rule and status settings for consumers to obtain and process messages in Apache RocketMQ.

Subscriptions are dynamically registered by consumer groups with brokers. Messages are then matched and consumed based on the filter rules defined by subscriptions.

By configuring subscriptions, you can control the following messaging behaviors:

- Message filter rules: These rules are used to define which messages in a topic are consumed by a consumer. By configuring message filter rules, consumers can effectively obtain messages that they want and specify message receiving ranges based on different business scenarios.

- Consumption status: By default, the Apache RocketMQ broker provides persistent subscriptions. In other words, after a consumer group subscribes to a broker, consumers in the group can continue consuming messages from where the consumers left off after they reconnect.

### Rules for determining a subscription

One topic to many subscribers. The following figure shows two consumer groups (Group A and Group B) subscribed to Topic A. These two subscriptions are independent of each other and can be defined separately.

![img](rocketmq-subscription.png)

One subscriber to multiple topics. The following figure shows a consumer group (Group A) subscribed to two topics: Topic A and Topic B. Consumers in Group A have two separate subscriptions to Topic A and Topic B. The two subscriptions are independent of each other and can be defined separately.

![img](rocketmq-subscription-one-subscriber-to-multiple-topics.png)

## Internal attributes[​](https://rocketmq.apache.org/docs/domainModel/09subscription#internal-attributes "Direct link to heading")

**Filter types**

- Definition: the type of a message filter rule. After a message filter rule is set for a subscription, the system matches the messages in a topic based on the filter rule. Only the messages that meet the conditions are delivered to consumers. This feature helps you classify messages sent to consumers based on your requirements.

- Values:
  
  - Tag filter: filters and matches the full text based on tag strings.
  
  - SQL92 filter: filters and matches message attributes based on SQL syntax.

**Filter expressions**

- Definition: the expression of a custom filter rule.

- Values: For more information, see [Syntax for filter expressions](https://rocketmq.apache.org/docs/featureBehavior/07messagefilter).

## Producer Group和Consumer Group对比

#### Producer Group

用来表示一个发送消息应用，一个 Producer Group 下包含多个 Producer 实例，可以是多台机器，也可以
是一台机器的多个进程，或者一个进程的多个 Producer 对象。一个 Producer Group 可以发送多个 Topic
消息，Producer Group 作用如下：

- 标识一类 Producer
- 可以通过运维工具查询这个发送消息应用下有多个 Producer 实例
- 发送分布式事务消息时，如果 Producer 中途意外宕机，Broker 会主动回调 Producer Group 内的任意
  一台机器来确认事务状态。

#### Consumer Group

用来表示一个消费消息应用，一个 Consumer Group 下包含多个 Consumer 实例，可以是多台机器，也可
以是多个进程，或者是一个进程的多个 Consumer 对象。一个 Consumer Group 下的多个 Consumer 以均摊
方式消费消息，如果设置为广播方式，那么这个 Consumer Group 下的每个实例都消费全量数据。

### 消费方式push和pull的区别

RocketMQ的是如何通过长轮询机制来实现压力和时实的平衡。

[RocketMQ的push消费方式实现的太聪明了-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2069437)

[【Alibaba中间件技术系列】「RocketMQ技术专题」让我们一起探索一下DefaultMQPullConsumer的实现原理及源码分析_阿里巴巴_洛神灬殇_InfoQ写作社区](https://xie.infoq.cn/article/2218308572d35f0a80682629c)

[精华推荐 | 【深入浅出 RocketMQ原理及实战】「底层源码挖掘系列」透彻剖析贯穿RocketMQ的消费者端的运行核心的流程（上篇）_RocketMQ_洛神灬殇_InfoQ写作社区](https://xie.infoq.cn/article/e3a0630707e4b7168aa13ea3c)

## 参考

https://rocketmq.apache.org/

[GitHub - apache/rocketmq-spring: Apache RocketMQ Spring Integration

](https://github.com/apache/rocketmq-spring)

[后端程序员必备：RocketMQ相关流程图/原理图 - 掘金](https://juejin.cn/post/6844903941839437838)
