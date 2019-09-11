---
title: Big Data框架--Kafka笔记
mathjax: false
date: 2019-09-08 09:48:40
tags:
- Big Data
- Kafka
categories:
- Big Data
top:
photo:
---



{% cq %}

**Kafka是一个分布式消息队列**

{% endcq %}

<!-- more -->

<br>

# 消息队列(MQ)

---

## 简介

![](https://i.loli.net/2019/09/08/cdZsvRw7fNYti2j.png)

- **点对点模式（一对一，消费者主动拉取数据，消息收到后消息清除）**

  点对点模型通常是一个基于拉取或者轮询的消息传送模型，这种模型从队列中请求信息，而不是将消息推送到客户端。这个模型的特点是发送到队列的消息被一个且只有一个接收者接收处理，即使有多个消息监听者也是如此。

- **发布/订阅模式（一对多，数据生产后，推送给所有订阅者）**

  发布订阅模型则是一个基于推送的消息传送模型。发布订阅模型可以有多种不同的订阅者，临时订阅者只在主动监听主题时才接收消息，而持久订阅者则监听主题的所有消息，即使当前订阅者不可用，处于离线状态。

## 优势

- **解耦**：

  允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

- **冗余**：

  消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

- **扩展性**：

  因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。

  

- **灵活性 & 峰值处理能力**：

  在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

  

- **可恢复性**：

  系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

- **顺序保证**：

  在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。（Kafka保证一个Partition内的消息的有序性）

- **缓冲**：

  有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。

- **异步通信**：

  很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

<br>

# 什么是kafka

---

在流式计算中，**Kafka一般用来缓存数据，Storm通过消费Kafka的数据进行计算**。

- Apache Kafka是一个开源**消息**系统，由Scala写成。是由Apache软件基金会开发的一个开源消息系统项目。
- Kafka最初是由LinkedIn公司开发，并于2011年初开源。2012年10月从Apache Incubator毕业。该项目的目标是为处理实时数据提供一个统一、高通量、低等待的平台。
- **Kafka是一个分布式消息队列**。Kafka对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接受者称为Consumer，此外kafka集群有多个kafka实例组成，每个实例(server)称为broker。
- 无论是kafka集群，还是consumer都依赖于**zookeeper**集群保存一些meta信息，来保证系统可用性。

<br>



# Kafka架构

---

![](https://i.loli.net/2019/09/08/bjGUrncHNsZlS7E.png)

- **Producer** ：消息生产者，就是向kafka broker**发消息**的客户端；

- **Consumer** ：消息消费者，向kafka broker**取消息**的客户端；

- **Topic** ：可以理解为一个**队列**；

- **Consumer Group （CG）**：这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。t**opic的消息会复制（不是真的复制，是概念上的）到所有的CG，但每个partion只会把消息发给该CG中的一个consumer。**如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic；

- **Broker** ：**一台kafka服务器**就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic；

- **Partition**：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序；

- **Offset**：kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找。例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafka。

<br>



# kafka生产过程分析

---

## 写入方式

producer采用**推（push）模式**将消息发布到broker，每条消息都被**追加（append）到分区（patition）中**，属于**顺序写磁盘**（顺序写磁盘效率比随机写内存要高，保障kafka吞吐率）。

## 分区（Partition）

**消息发送时都被发送到一个topic，其本质就是一个目录，而topic是由一些Partition Logs(分区日志)组成**，其组织结构如下图所示：

![](https://i.loli.net/2019/09/08/4ugoNCKyVbSMwQW.png)

**单个Partition内部流程**:

<img src="https://i.loli.net/2019/09/08/EXOxy9vwWr3zh8J.png"  />

**我们可以看到，每个Partition中的消息都是有序的，生产的消息被不断追加到Partition log上，其中的每一个消息都被赋予了一个唯一的offset值。**

### 分区的原因

- **方便在集群中扩展**，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了；

- **可以提高并发**，因为可以以Partition为单位读写了。

### 分区的原则

- **指定**了patition，则**直接使用**；

- 未指定patition但指定key，通过对key的value进行**hash**出一个patition；

- patition和key都未指定，使用**轮询**选出一个patition。

  源码分析:

  ```java
  DefaultPartitioner类
  public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
          List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
          int numPartitions = partitions.size();
          if (keyBytes == null) {
              int nextValue = nextValue(topic);
              List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
              if (availablePartitions.size() > 0) {
                  int part = Utils.toPositive(nextValue) % availablePartitions.size();
                  return availablePartitions.get(part).partition();
              } else {
                  // no partitions are available, give a non-available partition
                  return Utils.toPositive(nextValue) % numPartitions;
              }
          } else {
              // hash the keyBytes to choose a partition
              return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
          }
      }
  ```

  

## 副本（Replication）

**同一个partition可能会有多个replication**（对应 server.properties 配置中的 default.replication.factor=N）。没有replication的情况下，一旦broker 宕机，其上所有 patition 的数据都不可被消费，同时producer也不能再将数据存于其上的patition。**引入replication之后，同一个partition可能会有多个replication，而这时需要在这些replication之间选出一个leader，producer和consumer只与这个leader交互，其它replication作为follower从leader 中复制数据。**

## 写入流程

![](https://i.loli.net/2019/09/08/4iaXJMKYHvmhCyQ.png)

1. producer先从**zookeeper**的 "/brokers/.../state"节点**找到该partition的leader**

2. producer将消息**发送给该leader**

3. leader将消息**写入本地log**

4. followers从leader **pull消息**，写入本地log后**向leader发送ACK**

5. leader收到所有ISR中的replication的ACK后，**增加HW**（high watermark，最后commit 的offset）**并向producer发送ACK**

<br>

# **Broker 保存消息**

---

## **存储方式**

物理上把topic分成一个或多个patition（对应 server.properties 中的num.partitions=3配置），**每个patition物理上对应一个文件夹（该文件夹存储该patition的所有消息和索引文件）**，如下：

```shell
[@hadoop102 logs]$ ll
drwxrwxr-x. 2   4096 8月   6 14:37 first-0
drwxrwxr-x. 2   4096 8月   6 14:35 first-1
drwxrwxr-x. 2   4096 8月   6 14:37 first-2
[@hadoop102 logs]$ cd first-0
[@hadoop102 first-0]$ ll
-rw-rw-r--. 1  10485760 8月   6 14:33 00000000000000000000.index
-rw-rw-r--. 1       219 8月   6 15:07 00000000000000000000.log
-rw-rw-r--. 1  10485756 8月   6 14:33 00000000000000000000.timeindex
-rw-rw-r--. 1         8 8月   6 14:37 leader-epoch-checkpoint
```

## 存储策略

无论消息是否被消费，**kafka都会保留所有消息**。有两种策略可以删除旧数据：

- **基于时间**：log.retention.hours=168

- **基于大小**：log.retention.bytes=1073741824

> *需要注意的是，因为Kafka读取特定消息的时间复杂度为O(1)，即与文件大小无关，所以**这里删除过期文件与提高 Kafka 性能无关。***

## Zookeeper存储结构

![](https://i.loli.net/2019/09/08/NqDZ8b6uJKT7pzQ.png)

**注意：producer不在zk中注册，消费者在zk中注册。**

<br>

# Kafka消费过程分析

---

kafka提供了两套consumer API：高级Consumer API和低级Consumer API。

## 高级API

### **优点**

- 高级API 写起来简单

- 不需要自行去管理offset，系统通过zookeeper自行管理。

- 不需要管理分区，副本等情况，系统自动管理。

- **消费者断线会自动根据上一次记录在zookeeper中的offset去接着获取数据**（默认设置1分钟更新一下zookeeper中存的offset）

- 可以使用group来区分对同一个topic 的不同程序访问分离开来（**不同的group记录不同的offset，这样不同程序读取同一个topic才不会因为offset互相影响**）

### **缺点**

- 不能自行控制offset（对于某些特殊需求来说）

- 不能细化控制如分区、副本、zk等

## 低级API

### **优点**

- 能够让开发者**自己控制offset**，想从哪里读取就从哪里读取。

- **自行控制连接分区**，对分区自定义进行负载均衡

- **对zookeeper的依赖性降低**（如：offset不一定非要靠zk存储，自行存储offset即可，比如存在文件或者内存中）

### **缺点**

- 太过复杂，需要自行控制offset，连接哪个分区，找到分区leader 等。

## 消费者组

![](https://i.loli.net/2019/09/08/BoCijxKJStm2MXZ.png)

消费者是以consumer group消费者组的方式工作，**由一个或者多个消费者组成一个组，共同消费一个topic**。**每个分区在同一时间只能由group中的一个消费者读取，但是多个group可以同时消费这个partition**。在图中，有一个由三个消费者组成的group，有一个消费者读取主题中的两个分区，另外两个分别读取一个分区。某个消费者读取某个分区，也可以叫做某个消费者是某个分区的拥有者。

在这种情况下，**消费者可以通过水平扩展的方式同时读取大量的消息**。另外，如果一个消费者失败了，那么其他的group成员会**自动负载均衡读取之前失败的消费者读取的分区**。

## 消费方式

**consumer采用pull（拉）模式从broker中读取数据。**

**push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。**它的目标是尽可能以最快速度传递消息，但是这样很**容易造成consumer来不及处理消息**，典型的表现就是**拒绝服务以及网络拥塞**。而pull模式则可以根据consumer的消费能力以适当的速率消费消息。

对于Kafka而言，pull模式更合适，它**可简化broker的设计，consumer可自主控制消费消息的速率，同时consumer可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义。**

pull模式不足之处是，如果**kafka没有数据，消费者可能会陷入循环中，一直等待数据到达**。为了避免这种情况，我们在我们的拉请求中有参数，允许消费者请求在等待数据到达的“长轮询”中进行阻塞（并且可选地等待到给定的字节数，以确保大的传输大小）。

<br>

# **Kafka producer拦截器(interceptor)**

---

> *Producer拦截器(interceptor)是在Kafka 0.10版本被引入的，主要用于**实现clients端的定制化控制逻辑**。*

对于producer而言，interceptor使得用户在消息发送前以及producer回调逻辑前有机会对消息做一些定制化需求，比如修改消息等。同时，producer允许用户指定多个interceptor按序作用于同一条消息从而形成一个拦截链(interceptor chain)。Intercetpor的实现接口是org.apache.kafka.clients.producer.ProducerInterceptor，其定义的方法包括：

- **configure(configs)**

  **获取配置信息和初始化数据时调用。**

- **onSend(ProducerRecord)**：

  该方法封装进KafkaProducer.send方法中，即它运行在用户主线程中**。Producer确保在消息被序列化以及计算分区前调用该方法。**用户可以在该方法中对消息做任何操作，但最好保证不要修改消息所属的topic和分区，否则会影响目标分区的计算

- **onAcknowledgement(RecordMetadata, Exception)：**

  **该方法会在消息被应答或消息发送失败时调用**，并且通常都是在producer回调逻辑触发之前。onAcknowledgement运行在producer的IO线程中，因此不要在该方法中放入很重的逻辑，否则会拖慢producer的消息发送效率

- **close：**

  关闭interceptor，**主要用于执行一些资源清理工作**

  如前所述，interceptor可能被运行在多个线程中，因此在具体实现时用户需要自行确保线程安全。另外倘若指定了多个interceptor，则producer将按照指定顺序调用它们，并仅仅是捕获每个interceptor可能抛出的异常记录到错误日志中而非在向上传递。这在使用过程中要特别留意。

<br>

# **Kafka与Flume比较**

---

**flume：**

- cloudera公司研发
- 适合多个生产者；
- 适合下游数据消费者不多的情况；
- 适合数据安全性要求不高的操作；
- 适合与Hadoop生态圈对接的操作。

**kafka：**

- inkedin公司研发
- 适合数据下游消费众多的情况；
- 适合数据安全性要求较高的操作，支持replication。

**因此我们常用的一种模型是：**

**线上数据 --> flume --> kafka --> flume(根据情景增删该流程) --> HDFS**

<br>

# 总结

---

##  **Kafka压测**

Kafka官方自带压力测试脚本（kafka-consumer-perf-test.sh、kafka-producer-perf-test.sh）。Kafka压测时，可以查看到哪个地方出现了**瓶颈（CPU，内存，网络IO）。一般都是网络IO达到瓶颈。**

## **Kafka的机器数量**

Kafka机器数量=2*（峰值生产速度*副本数/100）+1

## Kafka的日志保存时间

7天

## **Kafka的硬盘大小**

每天的数据量*7天

## Kafka监控

公司自己开发的监控器；

开源的监控器：**KafkaManager、KafkaMonitor**

## Kakfa分区数

分区数并不是越多越好，**一般分区数不要超过集群机器数量**。分区数越多占用内存越大（ISR等），一个节点集中的分区也就越多，当它宕机的时候，对系统的影响也就越大。

**分区数一般设置为：3-10个**

在kafka中, topic的分区是并行计算的单元. 在producer端和broker端, 可以同时并发的写数据到不同的分区中. 在consumer端, Kafka总是将某个分区分配个一个consumer线程. 因此同一个消费组内的并行度与分区数息息相关.

Partition分区数的大小, 更多直接影响到消费端的吞吐(一个分区只能同一消费组的一个消费者消费). 分区数小, 消费端的吞吐就低. 但是太大也会有其他的影响

**原则:**

- 更多的分区可提高吞吐量
- 分区数越多打开的文件句柄越多
- 分区数越多降低可用性
- 更多的分区增加端到端的延迟
- 客户端需要更多的内存

归根结底还是得有个度. 如何找出这个度?

有个粗略的计算公式: `max(t/p, t/c)`. `t`就是**所预期吞吐量**, `p`是当**前生产端单个分区的吞吐**, 那`c`就是**消费端单个分区的吞吐.**

比如单个partition的生产端吞吐是200, 消费端是100. 预期的吞吐是500, 那么partition的数量就是5.

**单个分区的吞吐通常通过修改配置来提升, 比如生产端的批处理大小, 压缩算法, acknowledgement类型, 副本数等. 而在消费端则更依赖于消息的处理速度.**

## 副本数设定

一般我们设置成2个或3个，很多企业设置为2个。

## 多少个Topic

通常情况：**多少个日志类型就多少个Topic**。也有对日志类型进行合并的。

## Kafka的ACK数据备份

- Ack=0，相当于异步发送，消息发送完毕即offset增加，继续生产。

- Ack=1，leader收到leader replica 对一个消息的接受ack才增加offset，然后继续生产。

- Ack=-all，leader收到所有replica 对一个消息的接受ack才增加offset，然后继续生产。

## Kafka的ISR副本同步队列

ISR（In-Sync Replicas），副本同步队列。ISR中包括Leader和Follower。如果Leader进程挂掉，会在ISR队列中选择一个服务作为新的Leader。有replica.lag.max.messages（延迟条数）和replica.lag.time.max.ms（延迟时间）两个参数决定一台服务是否可以加入ISR副本队列，在0.10版本移除了replica.lag.max.messages参数，防止服务频繁的进去队列。

任意一个维度超过阈值都会把Follower剔除出ISR，存入OSR（Outof-Sync Replicas）列表，新加入的Follower也会先存放在OSR中。

## Kafka分区分配策略

在 Kafka内部存在两种默认的分区分配策略：**Range和 RoundRobin。**

### Range

**Range是默认策略**。Range是对每个Topic而言的（即一个Topic一个Topic分），首先对同一个Topic里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。然后用Partitions分区的个数除以消费者线程的总数来决定每个消费者线程消费几个分区。**如果除不尽，那么前面几个消费者线程将会多消费一个分区**。

例如：我们有10个分区，两个消费者（C1，C2），3个消费者线程，10 / 3 = 3而且除不尽。

C1-0 将消费 0, 1, 2, 3 分区

C2-0 将消费 4, 5, 6 分区

C2-1 将消费 7, 8, 9 分区

### RoundRobin

**前提：同一个Consumer Group里面的所有消费者的num.streams（消费者消费线程数）必须相等；每个消费者订阅的主题必须相同。**

将所有主题分区组成TopicAndPartition列表，然后对TopicAndPartition列表按照hashCode进行排序，最后按照轮询的方式发给每一个消费线程。

### Kafka中数据量计算

{% label danger @每天总数据量100g，每天产生1亿条日志， 10000万/24/60/60=1150条/每秒钟<br/>

平均每秒钟：1150条<br/>

低谷每秒钟：400条<br/>

高峰每秒钟：1150条*（2-20倍）=2300条-23000条<br/>

每条日志大小：0.5k-2k<br/>

每秒多少数据量：2.3M-20MB%}

### Kafka挂掉

- Flume记录

- 日志有记录

- 短期没事

### Kafka消息数据积压，Kafka消费能力不足怎么处理？

- **如果是Kafka消费能力不足，则可以考虑增加Topic的分区数**，并且同时提升消费组的消费者数量，消费者数=分区数。（两者缺一不可）

- 如果是下游的数据处理不及时：**提高每批次拉取的数量**。批次拉取数据过少（拉取数据/处理时间<生产速度），使处理的数据小于生产的数据，也会造成数据积压。