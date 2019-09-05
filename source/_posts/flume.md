---
title: Hadoop框架--Flume笔记
mathjax: false
date: 2019-09-05 14:47:26
tags:
- Big Data
- Flume
categories:
- Big Data
- Hadoop
top:
photo:
---



{% cq %}

Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统。Flume基于流式架构，灵活简单。

{% endcq %}

<!-- more -->

<br>

# 如何实现Flume数据传输的监控

---

- 使用第三方框架Ganglia实时监控Flume
- **中小企业监控平台选择 Zabbix**
- **互联网大企业监控平台选择 Ganglia+Centreon**

<br>

# Flume的Source，Sink，Channel的作用？

---

## 作用

- Source组件是专门用来收集数据的，可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy

- Channel组件对采集到的数据进行缓存，可以存放在Memory或File中。

- Sink组件是用于把数据发送到目的地的组件，目的地包括Hdfs、Logger、avro、thrift、ipc、file、Hbase、solr、自定义。

## 公司采用的Source类型为

- 监控后台日志：exec

- 监控后台产生日志的端口：netcat

<br>

# Flume参数调优

---

## Source

增加Source个（使用Tair Dir Source时可增加FileGroups个数）可以增大Source的读取数据的能力。例如：当某一个目录产生的文件过多时需要将这个文件目录拆分成多个文件目录，同时配置好多个Source 以保证Source有足够的能力获取到新产生的数据。

batchSize参数决定Source一次批量运输到Channel的event条数，适当调大这个参数可以提高Source搬运Event到Channel时的性能。

## Channel 

type 选择memory时Channel的性能最好，但是如果Flume进程意外挂掉可能会丢失数据。type选择file时Channel的容错性更好，但是性能上会比memory channel差。

使用file Channel时dataDirs配置多个不同盘下的目录可以提高性能。

Capacity 参数决定Channel可容纳最大的event条数。transactionCapacity 参数决定每次Source往channel里面写的最大event条数和每次Sink从channel里面读的最大event条数。transactionCapacity需要大于Source和Sink的batchSize参数。

## Sink 

增加Sink的个数可以增加Sink消费event的能力。Sink也不是越多越好够用就行，过多的Sink会占用系统资源，造成系统资源不必要的浪费。

batchSize参数决定Sink一次批量从Channel读取的event条数，适当调大这个参数可以提高Sink从Channel搬出event的性能。

<br>

# Flume最佳实践

---

- **模块命名规则**：所有的 Source 以 src 开头，所有的 Channel 以 ch 开头，所有的 Sink 以 sink 开头；

- 模块之间内部通信统一使用 Avro 接口；

- **将日志采集系统系统分为三层**：Agent 层，Collector 层和 Store 层，其中 Agent 层每个机器部署一个进程，负责对单机的日志收集工作；Collector 层部署在中心服务器上，负责接收 Agent 层发送的日志，并且将日志根据路由规则写到相应的 Store 层中；Store 层负责提供永久或者临时的日志存储服务，或者将日志流导向其它服务器。

- 扩展 MemoryChannel 和 FileChannel ，提供 DualChannel 的实现，以提供高吞吐和大缓存

- 监控 collector HdfsSink 写数据到 hdfs 的速度、FileChannel 中拥堵的 events 数量，以及写 hdfs 状态（查看是否有 .tmp 文件生成）

<br>

# **Flume的事务机制**

---

Flume的事务机制（类似数据库的事务机制）：**Flume使用两个独立的事务分别负责从Soucrce到Channel，以及从Channel到Sink的事件传递。**比如spooling directory source 为文件的每一行创建一个事件，一旦事务中所有的事件全部传递到Channel且提交成功，那么Soucrce就将该文件标记为完成。同理，事务以类似的方式处理从Channel到Sink的传递过程，如果因为某种原因使得事件无法记录，那么事务将会回滚。且所有的事件都会保持到Channel中，等待重新传递。

> *因此,Flume采集数据不会丢失,因为Channel存储可以在file中,数据传输有自己的事务管理*

<br>

# Flume Event 的批量处理

---

## 简介

在Flume可以批量处理事件。 Batch Size是Sink或者客户端单次事务可以从通道中取得事件的最大值。调低Batch Size参数，会造成吞吐量的降低，但同时也降低了失败时，发生事件多重记录的风险。 调高Batch Size参数，可以显著提高吞吐量，也会增加延迟，但是如果发生宕机，发生事件重复记录的风险也会相应的变高。

事务（Transaction）在Flume中是一个重要概念，因为只有在后面的每一个事务都成功的时候，可靠存储与送达才有保证。举个例子来说，当数据源接收到或者生产了一条事件， 为了将事件存储到通道，首先它必须先打开通道。在事务中， 数据源将Batch Size数量的事件送入通道中，然后才确认已经提交了事务。 Sink端需要经过同样的过程，才能从通道中提取事件。

Batch Size 是由Sink配置的。**Batch Size越大， Channel也就越快。** 文件通道的速度差异主要体现在它必须将缓冲区完全同步到硬盘时，事务才算成功提交。 硬盘同步本身就是一个非常耗时的操作（百毫秒级别），因为它要保证数据的可靠性。同样内存通道中每一个事务的提交也需要在内存中进行同步，不过在内存中这样的操作要比硬盘要快得多了。 如果你想了解更多文件通道的技术内幕，可以去看Brock Noland 的article about the Apache Flume File Channel。

使用大的Batch Size还有一个不好的地方，如果在事务传输中发生了失败，比如下游主机挂掉或者网络断开，会有一定的可能性发生数据重复。 举个例子，如果你的Batch Size是1000，而且你的下游机器离线了，那么可能就会发生大小为1000的数据重复。 这种情况的发生有几种可能，举个例子，如果事件已经写入了下游机器，在下游机器告知事务成功之前恰巧发生了连接断开，那么就会出现数据重复。然而你并不太需要忧心这一点，因为这只会发生在极其特殊的情况下。

## 最佳实践

- Sink的 Batch Size 应该等于输入流 Batch Size 的和。举个例子，如果你有一个 flume agent， 上游的10个Flume agent通过avro以100的batch Size 向这个agent发送事件，那么这个agent的batch Size 就应该设置为10X100 = 1000。
- 如果你发现你需要将Batch Size设置的非常大，比如比10000还大，那么需要考虑多设置几个channel来提高并发性（每一个channel都有一个自己的线程）。 实践中， 使用一个batch Size 为20000的HDFS Sink的性能要比 使用 两个batch Size 为10000的channel 性能要差。
- 在性能可接受的情况下，尽量选择较小的Batch Size。
- 为了做更详细的性能调优，要对channel做一些监控，例如Channel Size参数

