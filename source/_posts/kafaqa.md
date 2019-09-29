---
title: KafkaMQ--KafkaQA
mathjax: false
date: 2019-09-14 15:38:09
tags:
- Big Data
- Kafka
categories:
- Big Data
top:
photo:
---



{% cq %}

Kafka快速问答

{% endcq %}

<!-- more -->

<br>

# 请简述下你在哪些场景下会选择 Kafka？

---



- 日志收集：一个公司可以用 Kafka 可以收集各种服务的 log，通过 kafka 以统一接口服务的方式开放给各种 consumer，例如 hadoop、HBase、Solr 等。
- 消息系统：解耦和生产者和消费者、缓存消息等。
- 用户活动跟踪：Kafka 经常被用来记录 web 用户或者 app 用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到 kafka 的 topic 中，然后订阅者通过订阅这些 topic 来做实时的监控分析，或者装载到 hadoop、数据仓库中做离线分析和挖掘。
- 运营指标：Kafka 也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
- 流式处理：比如 spark streaming 和 Flink

<br>

# Kafka 分区的目的？

---

分区对于 Kafka 集群的好处是：**实现负载均衡**。

分区对于消费者来说，**可以提高并发度，提高效率**。

<br>

#  Kafka 是如何做到消息的有序性？

---

kafka 中的每个 partition 中的消息在写入时都是有序的，而且单独一个 partition 只能由一个消费者去消费，可以在里面保证消息的顺序性。但是分区之间的消息是不保证有序的。

<br>

# Kafka分区分配策略(Partition Assignment Strategy)

---

为了使得我们能够及时消费消息，我们也可能会启动多个 Consumer 去消费，而每个 Consumer 又会启动一个或多个streams去分别消费 Topic 里面的数据。我们又知道，Kafka 存在 Consumer Group 的概念，也就是 group.id 一样的 Consumer，这些 Consumer 属于同一个Consumer Group，组内的所有消费者协调在一起来消费订阅主题(subscribed topics)的所有分区(partition)。当然，每个分区只能由同一个消费组内的一个consumer来消费。那么问题来了，**同一个 Consumer Group 里面的 Consumer 是如何知道该消费哪些分区里面的数据呢？**

![](https://i.loli.net/2019/09/14/7YHa6US1bpR4PG9.png)

如上图，Consumer1 为啥消费的是 Partition0 和 Partition2，而不是 Partition0 和 Partition3？这就涉及到 Kafka 内部分区分配策略（Partition Assignment Strategy）了。

在 Kafka 内部存在两种默认的分区分配策略：Range 和 RoundRobin。当以下事件发生时，Kafka 将会进行一次分区分配：

- 同一个 Consumer Group 内新增消费者
- 消费者离开当前所属的Consumer Group，包括shuts down 或 crashes
- 订阅的主题新增分区

将分区的所有权从一个消费者移到另一个消费者称为重新平衡（rebalance），如何rebalance就涉及到本文提到的分区分配策略。下面我们将详细介绍 Kafka 内置的两种分区分配策略。本文假设我们有个名为 T1 的主题，其包含了10个分区，然后我们有两个消费者（C1，C2）来消费这10个分区里面的数据，而且 C1 的 num.streams = 1，C2 的 num.streams = 2。

## RANGE STRATEGY

Range策略是对每个主题而言的，首先对同一个主题里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。在我们的例子里面，排完序的分区将会是0, 1, 2, 3, 4, 5, 6, 7, 8, 9；消费者线程排完序将会是C1-0, C2-0, C2-1。然后将partitions的个数除于消费者线程的总数来决定每个消费者线程消费几个分区。如果除不尽，那么前面几个消费者线程将会多消费一个分区。在我们的例子里面，我们有10个分区，3个消费者线程， 10 / 3 = 3，而且除不尽，那么消费者线程 C1-0 将会多消费一个分区，所以最后分区分配的结果看起来是这样的：

```
C1-0 将消费 0, 1, 2, 3 分区
C2-0 将消费 4, 5, 6 分区
C2-1 将消费 7, 8, 9 分区
```

假如我们有11个分区，那么最后分区分配的结果看起来是这样的：

```
C1-0 将消费 0, 1, 2, 3 分区
C2-0 将消费 4, 5, 6, 7 分区
C2-1 将消费 8, 9, 10 分区
```

假如我们有2个主题(T1和T2)，分别有10个分区，那么最后分区分配的结果看起来是这样的：

```
C1-0 将消费 T1主题的 0, 1, 2, 3 分区以及 T2主题的 0, 1, 2, 3分区
C2-0 将消费 T1主题的 4, 5, 6 分区以及 T2主题的 4, 5, 6分区
C2-1 将消费 T1主题的 7, 8, 9 分区以及 T2主题的 7, 8, 9分区
```

可以看出，C1-0 消费者线程比其他消费者线程多消费了2个分区，这就是Range strategy的一个很明显的弊端。


  ## RoundRobin strategy

使用RoundRobin策略有两个前提条件必须满足：

- 同一个Consumer Group里面的所有消费者的num.streams必须相等；
- 每个消费者订阅的主题必须相同。

所以这里假设前面提到的2个消费者的num.streams = 2。RoundRobin策略的工作原理：将所有主题的分区组成 TopicAndPartition 列表，然后对 TopicAndPartition 列表按照 hashCode 进行排序，这里文字可能说不清，看下面的代码应该会明白：

```scala
val allTopicPartitions = ctx.partitionsForTopic.flatMap { case(topic, partitions) =>
  info("Consumer %s rebalancing the following partitions for topic %s: %s"
       .format(ctx.consumerId, topic, partitions))
  partitions.map(partition => {
    TopicAndPartition(topic, partition)
  })
}.toSeq.sortWith((topicPartition1, topicPartition2) => {
  /*
   * Randomize the order by taking the hashcode to reduce the likelihood of all partitions of a given topic ending
   * up on one consumer (if it has a high enough stream count).
   */
  topicPartition1.toString.hashCode < topicPartition2.toString.hashCode
})
```

最后按照round-robin风格将分区分别分配给不同的消费者线程。

在我们的例子里面，加入按照 hashCode 排序完的topic-partitions组依次为T1-5, T1-3, T1-0, T1-8, T1-2, T1-1, T1-4, T1-7, T1-6, T1-9，我们的消费者线程排序为C1-0, C1-1, C2-0, C2-1，最后分区分配的结果为：

```
C1-0 将消费 T1-5, T1-2, T1-6 分区；
C1-1 将消费 T1-3, T1-1, T1-9 分区；
C2-0 将消费 T1-0, T1-4 分区；
C2-1 将消费 T1-8, T1-7 分区；
```

根据上面的详细介绍相信大家已经对Kafka的分区分配策略原理很清楚了。不过遗憾的是，**目前我们还不能自定义分区分配策略**，只能通过partition.assignment.strategy参数选择 range 或 roundrobin。partition.assignment.strategy参数默认的值是range。

<br>

# Kafka如何实现高吞吐的

---

Kafka 是分布式消息系统，需要处理海量的消息，Kafka 的设计是把所有的消息都写入速度低容量大的硬盘，以此来换取更强的存储能力，但实际上，使用硬盘并没有带来过多的性能损失
kafka 主要使用了以下几个方式实现了超高的吞吐率

## 顺序读写

kafka 的消息是不断追加到文件中的，这个特性使 kafka 可以充分利用磁盘的顺序读写性能
顺序读写不需要硬盘磁头的寻道时间，只需很少的扇区旋转时间，所以速度远快于随机读写
Kafka 官方给出了测试数据 (Raid-5，7200rpm)：
顺序 I/O: 600MB/s
随机 I/O: 100KB/s

## 零拷贝

先简单了解下文件系统的操作流程，例如一个程序要把文件内容发送到网络
这个程序是工作在用户空间，文件和网络 socket 属于硬件资源，两者之间有一个内核空间
在操作系统内部，整个过程为：

![](https://i.loli.net/2019/09/14/bU2eDEA7Lvs6TZz.png)

在 Linux kernel2.2 之后出现了一种叫做 "零拷贝 (zero-copy)" 系统调用机制，就是跳过“用户缓冲区” 的拷贝，建立一个磁盘空间和内存的直接映射，数据不再复制到“用户态缓冲区”
系统上下文切换减少为 2 次，可以提升一倍的性能

![](https://i.loli.net/2019/09/14/8EyXxeAPrsVOdwz.png)

## 文件分段

kafka 的队列 topic 被分为了多个区 partition，每个 partition 又分为多个段 segment，所以一个队列中的消息实际上是保存在 N 多个片段文件中

![](https://i.loli.net/2019/09/14/mcfqaMSt78d2CZ1.png)

通过分段的方式，每次文件操作都是对一个小文件的操作，非常轻便，同时也增加了并行处理能力

## 批量发送

**Kafka 允许进行批量发送消息，先将消息缓存在内存中，然后一次请求批量发送出去**
比如可以指定缓存的消息达到某个量的时候就发出去，或者缓存了固定的时间后就发送出去
如 100 条消息就发送，或者每 5 秒发送一次
这种策略将大大减少服务端的 I/O 次数

## 数据压缩

**Kafka 还支持对消息集合进行压缩，Producer 可以通过 GZIP 或 Snappy 格式对消息集合进行压缩**
压缩的好处就是减少传输的数据量，减轻对网络传输的压力
Producer 压缩之后，在 Consumer 需进行解压，虽然增加了 CPU 的工作，但在对大数据处理上，瓶颈在网络上而不是 CPU，所以这个成本很值得

<br>



# Kafka 是如何保证数据可靠性的

---

## Topic 分区副本

在 Kafka 0.8.0 之前，Kafka 是没有副本的概念的，那时候人们只会用 Kafka 存储一些不重要的数据，因为没有副本，数据很可能会丢失。但是随着业务的发展，支持副本的功能越来越强烈，所以为了保证数据的可靠性，Kafka 从 0.8.0 版本开始引入了分区副本（详情请参见 [KAFKA-50](https://www.iteblog.com/redirect.php?url=aHR0cHM6Ly9pc3N1ZXMuYXBhY2hlLm9yZy9qaXJhL2Jyb3dzZS9LQUZLQS01MA==&article=true)）。也就是说**每个分区可以人为的配置几个副本（比如创建主题的时候指定 `replication-factor`，也可以在 Broker 级别进行配置 `default.replication.factor`），一般会设置为3**。

**Kafka 可以保证单个分区里的事件是有序的**，分区可以在线（可用），也可以离线（不可用）。在众多的分区副本里面有一个副本是 Leader，其余的副本是 follower，所有的读写操作都是经过 Leader 进行的，同时 follower 会定期地去 leader 上的复制数据。当 Leader 挂了的时候，其中一个 follower 会重新成为新的 Leader。通过分区副本，引入了数据冗余，同时也提供了 Kafka 的数据可靠性。

**Kafka 的分区多副本架构是 Kafka 可靠性保证的核心，把消息写入多个副本可以使 Kafka 在发生崩溃时仍能保证消息的持久性。**

## Producer 往 Broker 发送消息

如果我们要往 Kafka 对应的主题发送消息，我们需要通过 Producer 完成。前面我们讲过 Kafka 主题对应了多个分区，每个分区下面又对应了多个副本；为了让用户设置数据可靠性， Kafka 在 Producer 里面提供了消息确认机制。也就是说**我们可以通过配置来决定消息发送到对应分区的几个副本才算消息发送成功**。可以在定义 Producer 时通过 `acks` 参数指定（在 0.8.2.X 版本之前是通过 `request.required.acks` 参数设置的，详见 [KAFKA-3043](https://www.iteblog.com/redirect.php?url=aHR0cHM6Ly9pc3N1ZXMuYXBhY2hlLm9yZy9qaXJhL2Jyb3dzZS9LQUZLQS0zMDQz&article=true)）。这个参数支持以下三种值：

- **acks = 0**：意味着如果生产者能够通过网络把消息发送出去，那么就认为消息已成功写入 Kafka 。在这种情况下还是有可能发生错误，比如发送的对象无能被序列化或者网卡发生故障，但如果是分区离线或整个集群长时间不可用，那就不会收到任何错误。在 acks=0 模式下的运行速度是非常快的（这就是为什么很多基准测试都是基于这个模式），你可以得到惊人的吞吐量和带宽利用率，不过如果选择了这种模式， 一定会丢失一些消息。
- **acks = 1**：意味若 Leader 在收到消息并把它写入到分区数据文件（不一定同步到磁盘上）时会返回确认或错误响应。在这个模式下，如果发生正常的 Leader 选举，生产者会在选举时收到一个 LeaderNotAvailableException 异常，如果生产者能恰当地处理这个错误，它会重试发送悄息，最终消息会安全到达新的 Leader 那里。不过在这个模式下仍然有可能丢失数据，比如消息已经成功写入 Leader，但在消息被复制到 follower 副本之前 Leader发生崩溃。
- **acks = al**l（这个和 request.required.acks = -1 含义一样）：意味着 Leader 在返回确认或错误响应之前，会等待所有同步副本都收到悄息。如果和 `min.insync.replicas` 参数结合起来，就可以决定在返回确认前至少有多少个副本能够收到悄息，生产者会一直重试直到消息被成功提交。不过这也是最慢的做法，因为生产者在继续发送其他消息之前需要等待所有副本都收到当前的消息。

根据实际的应用场景，我们设置不同的 `acks`，以此保证数据的可靠性。

另外，Producer 发送消息还可以选择同步（默认，通过 `producer.type=sync` 配置） 或者异步（`producer.type=async`）模式。如果设置成异步，虽然会极大的提高消息发送的性能，但是这样会增加丢失数据的风险。如果需要确保消息的可靠性，必须将 `producer.type` 设置为 sync。

## Leader 选举

每个分区的 leader 会维护一个 ISR 列表，ISR 列表里面就是 follower 副本的 Borker 编号，只有跟得上 Leader 的 follower 副本才能加入到 ISR 里面，这个是通过 `replica.lag.time.max.ms` 参数配置的，只有 ISR 里的成员才有被选为 leader 的可能。

所以当 Leader 挂掉了，而且 `unclean.leader.election.enable=false` 的情况下，Kafka 会从 ISR 列表中选择第一个 follower 作为新的 Leader，因为这个分区拥有最新的已经 committed 的消息。通过这个可以保证已经 committed 的消息的数据可靠性。

综上所述，为了保证数据的可靠性，我们最少需要配置一下几个参数：

- **producer 级别：acks=all（或者 request.required.acks=-1），同时发生模式为同步 producer.type=sync**
- **topic 级别：设置 replication.factor>=3，并且 min.insync.replicas>=2；**
- **broker 级别：关闭不完全的 Leader 选举，即 unclean.leader.election.enable=false；**

<br>

# Kafka是如何保证数据一致性的

---

这里介绍的数据一致性主要是说不论是老的 Leader 还是新选举的 Leader，Consumer 都能读到一样的数据。那么 Kafka 是如何实现的呢？

![](C:\Users\Administrator\Desktop\big data\xkFyjVo8ngTHKeU.png)

假设分区的副本为3，其中副本0是 Leader，副本1和副本2是 follower，并且在 ISR 列表里面。虽然副本0已经写入了 Message4，但是 Consumer 只能读取到 Message2。因为所有的 ISR 都同步了 Message2，**只有 High Water Mark 以上的消息才支持 Consumer 读取**，而 High Water Mark 取决于 ISR 列表里面偏移量最小的分区，对应于上图的副本2，这个很类似于木桶原理。

这样做的原因是还没有被足够多副本复制的消息被认为是“不安全”的，如果 Leader 发生崩溃，另一个副本成为新 Leader，那么这些消息很可能丢失了。如果我们允许消费者读取这些消息，可能就会破坏一致性。试想，一个消费者从当前 Leader（副本0） 读取并处理了 Message4，这个时候 Leader 挂掉了，选举了副本1为新的 Leader，这时**候另一个消费者再去从新的 Leader 读取消息，发现这个消息其实并不存在，这就导致了数据不一致性问题**。

当然，引入了 High Water Mark 机制，会导致 Broker 间的消息复制因为某些原因变慢，那么消息到达消费者的时间也会随之变长（因为我们会先等待消息复制完毕）。延迟时间可以通过参数 `replica.lag.time.max.ms` 参数配置，它**指定了副本在复制消息时可被允许的最大延迟时间**。

<br>

# ISR、OSR、AR 是什么？

---

**ISR：In-Sync Replicas 副本同步队列**

**OSR：Out-of-Sync Replicas** 

**AR：Assigned Replicas 所有副本**

ISR 是由 leader 维护，follower 从 leader 同步数据有一些延迟，超过相应的阈值会把 follower 剔除出 ISR, 存入 OSR（Out-of-Sync Replicas ）列表，新加入的 follower 也会先存放在 OSR 中。**AR=ISR+OSR。**

<br>

# LEO、HW、LSO、LW 等分别代表什么?

- **LEO**：是 LogEndOffset 的简称，**代表当前日志文件中下一条**
- **HW**：**水位或水印（watermark）一词**，也可称为高水位 (high watermark)，通常被用在流式处理领域（比如 Apache Flink、Apache Spark 等），以表征元素或事件在基于时间层面上的进度。在 Kafka 中，水位的概念反而与时间无关，而是与位置信息相关。严格来说，**它表示的就是位置信息，即位移（offset）。取 partition 对应的 ISR 中 最小的 LEO 作为 HW，consumer 最多只能消费到 HW 所在的位置上一条信息。**
- **LSO**：是 LastStableOffset 的简称，**对未完成的事务而言，LSO 的值等于事务中第一条消息的位置 (firstUnstableOffset)**，对已完成的事务而言，它的值同 HW 相同 
- **LW**：Low Watermark 低水位, **代表 AR 集合中最小的 logStartOffset 值**。

<br>

# Kafka 消费者是否可以消费指定分区消息？

---

Kafa consumer 消费消息时，向 broker 发出 fetch 请求去消费特定分区的消息，consumer 指定消息在日志中的偏移量（offset），就可以消费从这个位置开始的消息，customer 拥有了 offset 的控制权，可以向后回滚去重新消费之前的消息

<br>

# Kafka 消息是采用 Pull 模式，还是 Push 模式？

---

Kafka 最初考虑的问题是，customer 应该从 brokes 拉取消息还是 brokers 将消息推送到 consumer，也就是 pull 还 push。在这方面，Kafka 遵循了一种大部分消息系统共同的传统的设计：**producer 将消息推送到 broker，consumer 从 broker 拉取消息**。

一些消息系统比如 **Scribe 和 Apache Flume 采用了 push 模式**，将消息推送到下游的 consumer。这样做有好处也有坏处：由 broker 决定消息推送的速率，对于不同消费速率的 consumer 就不太好处理了。消息系统都致力于让 consumer 以最大的速率最快速的消费消息，但不幸的是，**push 模式下，当 broker 推送的速率远大于 consumer 消费的速率时，consumer 恐怕就要崩溃了**。最终 Kafka 还是选取了传统的 pull 模式。

**Pull 模式的另外一个好处是 consumer 可以自主决定是否批量的从 broker 拉取数据**。Push 模式必须在不知道下游 consumer 消费能力和消费策略的情况下决定是立即推送每条消息还是缓存之后批量推送。如果为了避免 consumer 崩溃而采用较低的推送速率，将可能导致一次只推送较少的消息而造成浪费。Pull 模式下，consumer 就可以根据自己的消费能力去决定这些策略。

Pull 有个缺点是，**如果 broker 没有可供消费的消息，将导致 consumer 不断在循环中轮询，直到新消息到 t 达**。为了避免这点，Kafka 有个参数可以让 consumer 阻塞知道新消息到达 (当然也可以阻塞知道消息的数量达到某个特定的量这样就可以批量发

<br>

# Kafka 高效文件存储设计特点

---

- **Kafka 把 topic 中一个 parition 大文件分成多个小文件段**，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用。
- 通过**索引信息**可以快速定位 message 和确定 response 的最大大小。
- 通过 index **元数据全部映**射到 memory，可以避免 segment file 的 IO 磁盘操作。
- 通过**索引文件稀疏存储**，可以大幅降低 index 文件元数据占用空间大小

<br>

# Kafka 创建 Topic 时如何将分区放置到不同的 Broker 中

---

- **副本因子不能大于 Broker 的个数**；
- 第一个分区（编号为 0）的第一个副本放置位置是随机从 brokerList 选择的；
- 其他分区的第一个副本放置位置相对于第 0 个分区依次往后移。也就是如果**我们有 5 个 Broker，5 个分区，假设第一个分区放在第四个 Broker 上，那么第二个分区将会放在第五个 Broker 上；第三个分区将会放在第一个 Broker 上；第四个分区将会放在第二个 Broker 上，依次类推**；
- 剩余的副本相对于第一个副本放置位置其实是由 **nextReplicaShift** 决定的，而这个数也是随机产生的

<br>

# 谈一谈 Kafka 的再均衡

---

在 Kafka 中，**当有新消费者加入或者订阅的 topic 数发生变化时**，会触发 Rebalance(再均衡：在同一个消费者组当中，分区的所有权从一个消费者转移到另外一个消费者) 机制，Rebalance 顾名思义就是重新均衡消费者消费。Rebalance 的过程如下：

- **第一步**：**所有成员都向 coordinator 发送请求，请求入组**。一旦所有成员都发送了请求，coordinator 会从中选择一个 consumer 担任 leader 的角色，并把组成员信息以及订阅信息发给 leader。

- **第二步**：**leader 开始分配消费方案，指明具体哪个 consumer 负责消费哪些 topic 的哪些 partition**。一旦完成分配，leader 会将这个方案发给 coordinator。coordinator 接收到分配方案之后会把方案发给各个 consumer，这样组内的所有成员就都知道自己应该消费哪些分区了。

**所以对于 Rebalance 来说，Coordinator 起着至关重要的作用**

<br>

# Kafka 分区数可以增加或减少吗？为什么？

---

**我们可以使用 bin/kafka-topics.sh 命令对 Kafka 增加 Kafka 的分区数据，但是 Kafka 不支持减少分区数。** 

Kafka 分区数据不支持减少是由很多原因的，比如减少的分区其数据放到哪里去？是删除，还是保留？删除的话，那么这些没消费的消息不就丢了。如果保留这些消息如何放到其他分区里面？追加到其他分区后面的话那么就破坏了 Kafka 单个分区的有序性。如果要保证删除分区数据插入到其他分区保证有序性，那么实现起来逻辑就会非常复杂。

<br>

# **Kafka 重复消费同一 Topic 数据**

---

## 在 **高级API** 中，消费者要从头开始消费某个 **topic** 的全量数据，需要满足 2 个条件：


- 使用一个全新的"group.id"（就是之前没有被任何消费者使用过）;

- 使用assign来订阅；

注意：如果把 **"enable.auto.commit" 设为 "false"**，使用 **consumer.commitAsync(currentOffsets, null)** 手动提交 **offset** ，是不能从头开始消费的

## **auto.offset.reset 值含义解释**:

**earliest**:
当各分区下有已提交的 offset 时，从提交的 offset 开始消费；无提交的 offset 时，从头开始消费
**latest**:
当各分区下有已提交的 offset 时，从提交的 offset 开始消费；无提交的 offset 时，消费新产生的该分区下的数据
**none**:
topic 各分区都存在已提交的 offset 时，从 offset 后开始消费；只要有一个分区不存在已提交的 offset，则抛出异常

> 也就是说无论哪种设置，只要 **kafka** 中相同 **group**、**partition** 中已经有提交的 **offset**，则都无法从开始消费。

## API解释:(high/low level)

**KafkaConsumer.subscribe() :** 

为consumer自动分配partition，有内部算法保证topic-partition以最优的方式均匀分配给同group下的不同consumer。如果有多个partition且只有一个消费者，则按顺序消费所有分区。不会重复消费。

**KafkaConsumer.assign() :**

 为consumer手动、显示的指定需要消费的topic-partitions，不受group.id限制，不提交offset，相当与指定的group无效（this method does not use the consumer's group management）。可以重复消费。





**转载自过往记忆（https://www.iteblog.com/）**