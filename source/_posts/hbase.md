---
title: Big Data框架--HBase笔记
mathjax: false
date: 2019-09-08 19:06:23
tags:
- Big Data
- HBase
categories:
- Big Data
top:
photo:
---



{% cq %}

HBase是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用HBASE技术可在廉价PC Server上搭建起大规模结构化存储集群。

{% endcq %}

<!-- more -->

<br>

# Hbase简介

---

##  HBase特点

### 海量存储

**Hbase适合存储PB级别的海量数据**，在PB级别的数据以及采用廉价PC存储的情况下，能在几十到百毫秒内返回数据。这与Hbase的极易扩展性息息相关。正式因为Hbase良好的扩展性，才为海量数据的存储提供了便利。

### 列式存储

这里的列式存储其实说的是列族存储，Hbase是根据列族来存储数据的。列族下面可以有非常多的列，列族在创建表的时候就必须指定。

### 极易拓展

Hbase的扩展性主要体现在两个方面:

- **基于上层处理能力（RegionServer）的扩展**
- **基于存储的扩展（HDFS）**。

通过横向添加RegionSever的机器，进行水平扩展，提升Hbase上层的处理能力，提升Hbsae服务更多Region的能力。

> **备注：**RegionServer的作用是管理region、承接业务的访问，通过横向添加Datanode的机器，进行存储层扩容，提升Hbase的数据存储能力和提升后端存储的读写能力。

### 高并发

由于目前大部分使用Hbase的架构，都是采用的廉价PC，因此单个IO的延迟其实并不小，一般在几十到上百ms之间。这里说的高并发，**主要是在并发的情况下，Hbase的单个IO延迟下降并不多。能获得高并发、低延迟的服务**。

### 稀疏

稀疏主要是针对Hbase列的灵活性,在列族中你可以指定任意多的列,在列数据为空的情况下,是不会占用存储空间的

## Hbase架构

如图:

![](https://i.loli.net/2019/09/08/EiCDFcAqrRyYGMj.png)

从图中可以看出Hbase是由**Client、Zookeeper、Master、HRegionServer、HDFS**等几个组件组成

### Client

Client包含了**访问Hbase的接口**，另外**Client还维护了对应的cache来加速Hbase的访问**，比如cache的.META.元数据的信息。

### **Zookeeper**

HBase通过Zookeeper来做**master的高可用、RegionServer的监控、元数据的入口以及集群配置的维护**等工作。具体工作如下：

- 通过Zoopkeeper来保证集群中只有1个master在运行，如果master异常，会通过竞争机制产生新的master提供服务

- 通过Zoopkeeper来监控RegionServer的状态，当RegionSevrer有异常的时候，通过回调的形式通知Master RegionServer上下线的信息

- 通过Zoopkeeper存储元数据的统一入口地址

### Hmaster

master节点的主要职责如下：

- 为RegionServer**分配Region**
- 维护整个集群的**负载均衡**
- 维护集群的**元数据**信息
- **发现失效的Region**，并将失效的Region分配到正常的RegionServer上
- 当RegionSever失效的时候，**协调对应Hlog的拆分**

### **HregionServer**

HregionServer**直接对接用户的读写请求，是真正的“干活”的节点**。它的功能概括如下：

- 管理master为其分配的Region
- 处理来自客户端的**读写请求**
- 负责和**底层HDFS的交互**，存储数据到HDFS
- 负责Region变大以后的**拆分**
- 负责**Storefile的合并**工作

### HDFS

HDFS**为Hbase提供最终的底层数据存储服务，同时为HBase提供高可用（Hlog存储在HDFS）的支持**，具体功能概括如下：

- **提供元数据和表数据的底层分布式存储服务**
- **数据多副本，保证的高可靠和高可用性**



## HBase中的角色

### HMaster

**功能**

1. 监控RegionServer
2. 处理RegionServer故障转移
3. 处理元数据的变更
4. 处理region的分配或转移
5. 在空闲时间进行数据的负载均衡
6. 通过Zookeeper发布自己的位置给客户端

### RegionServer

**功能**

1. 负责存储HBase的实际数据
2. 处理分配给它的Region
3. 刷新缓存到HDF
4. 维护Hlog
5. 执行压缩
6. 负责处理Region分片

### 其他组件

**1．Write-Ahead logs**

HBase的修改记录，当对HBase读写数据的时候，数据不是直接写进磁盘，它会在内存中保留一段时间（时间以及数据量阈值可以设定）。但把数据保存在内存中可能有更高的概率引起数据丢失，为了解决这个问题，数据会先写在一个叫做Write-Ahead logfile的文件中，然后再写入内存中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。

**2．Region**

Hbase表的分片，HBase表会根据RowKey值被切分成不同的region存储在RegionServer中，在一个RegionServer中可以有多个不同的region。

**3．Store**

HFile存储在Store中，一个Store对应HBase表中的一个列族。

**4．MemStore**

顾名思义，就是内存存储，位于内存中，用来保存当前的数据操作，所以当数据保存在WAL中之后，RegsionServer会在内存中存储键值对。

**5．HFile**

这是在磁盘上保存原始数据的实际的物理文件，是实际的存储文件。StoreFile是以Hfile的形式存储在HDFS的。

<br>

# **HBase数据结构**

---

## RowKey

与nosql数据库们一样,**RowKey是用来检索记录的主键**。访问HBASE table中的行，只有三种方式：

- **通过单个RowKey访问**
- **通过RowKey的range（正则）**
- **全表扫描**

RowKey行键 (RowKey)可以是任意字符串(最大长度是64KB，实际应用中长度一般为 10-100bytes)，在HBASE内部，**RowKey保存为字节数组。存储时，数据按照RowKey的字典序(byte order)排序存储**。设计RowKey时，要充分排序存储这个特性，将经常一起读取的行存储放到一起。(位置相关性)

## **Column Family**

列族：**HBASE表中的每个列，都归属于某个列族**。

**列族是表的schema的一部 分(而列不是)，必须在使用表之前定义**。列名都以列族作为前缀。例如 courses:history，courses:math都属于courses 这个列族。

## Cell

由{rowkey, column Family:columu, version} 唯一确定的单元。cell中的数据是没有类型的，全部是字节码形式存贮。

**关键字：无类型、字节码**

## Time Stamp

HBASE 中通过rowkey和columns确定的为一个存贮单元称为cell。每个 cell都保存 着同一份数据的多个版本。版本通过时间戳来索引。时间戳的类型是 64位整型。时间戳可以由HBASE(在数据写入时自动 )赋值，此时时间戳是精确到毫秒 的当前系统时间。时间戳也可以由客户显式赋值。如果应用程序要避免数据版 本冲突，就必须自己生成具有唯一性的时间戳。每个 cell中，不同版本的数据按照时间倒序排序，即最新的数据排在最前面。

为了避免数据存在过多版本造成的的管理 (包括存贮和索引)负担，HBASE提供 了两种数据版本回收方式:

**一是保存数据的最后n个版本，二是保存最近一段 时间内的版本（比如最近七天）。用户可以针对每个列族进行设置。**

## 命名空间

![](https://i.loli.net/2019/09/08/deb5RoSc6yTN3fC.png)

**Table**：表，所有的表都是命名空间的成员，即表必属于某个命名空间，如果没有指定，则在default默认的命名空间中。

**RegionServer group**：一个命名空间包含了默认的RegionServer Group。

**Permission**：权限，命名空间能够让我们来定义访问控制列表ACL（Access Control List）。例如，创建表，读取表，删除，更新等等操作。

**Quota**：限额，可以强制一个命名空间可包含的region的数量。

<br>

# Hbase原理

---

## 读流程

![](https://i.loli.net/2019/09/08/qCQbjWuAhRGYmP9.png)

1. Client先访问zookeeper，从meta表读取region的位置，然后读取meta表中的数据。meta中又存储了用户表的region信息；
2. 根据namespace、表名和rowkey在meta表中找到对应的region信息；
3. 找到这个region对应的regionserver；
4. 查找对应的region；
5. 先从MemStore找数据，如果没有，再到BlockCache里面读；
6. lockCache还没有，再到StoreFile上读(为了读取的效率)；
7. 如果是从StoreFile里面读取的数据，不是直接返回给客户端，而是先写入BlockCache，再返回给客户端。

## 写流程

![](https://i.loli.net/2019/09/08/Az3Br4VG9hCgyZb.png)

1. Client向HregionServer发送写请求；
2. HregionServer将数据写到HLog（write ahead log）。为了数据的持久化和恢复；
3. HregionServer将数据写到内存（MemStore）；
4. 反馈Client写成功。

## 数据Flush过程

1. 当MemStore数据达到阈值（默认是128M，老版本是64M），将数据刷到硬盘，将内存中的数据删除，同时删除HLog中的历史数据；
2. 并将数据存储到HDFS中；
3. 在HLog中做标记点。

## 数据合并过程

1. 当数据块达到4块，Hmaster触发合并操作，Region将数据块加载到本地，进行合并；
2. 当合并的数据超过256M，进行拆分，将拆分后的Region分配给不同的HregionServer管理；
3. 当HregionServer宕机后，将HregionServer上的hlog拆分，然后分配给不同的HregionServer加载，修改.META.；
4. 注意：HLog会同步到HDFS。

<br>

# Hbase优化

---

## 高可用

在HBase中Hmaster负责监控RegionServer的生命周期，均衡RegionServer的负载，如果Hmaster挂掉了，那么整个HBase集群将陷入不健康的状态，并且此时的工作状态并不会维持太久。所以HBase支持对Hmaster的高可用配置。

1. 关闭HBase集群（如果没有开启则跳过此步）

   `[hadoop102 hbase]$ bin/stop-hbase.sh`

2. 在conf目录下创建backup-masters文件

   `[hadoop102 hbase]$ touch conf/backup-masters`

3. 在backup-masters文件中配置高可用HMaster节点

   `[hadoop102 hbase]$ echo hadoop103 > conf/backup-masters`

4. 将整个conf目录scp到其他节点

   `[hadoop102 hbase]$ scp -r conf/ hadoop103:/opt/module/hbase/`

   `[hadoop102 hbase]$ scp -r conf/ hadoop104:/opt/module/hbase/`

##  HBase的rowkey的设计原则

HBase是三维有序存储的，通过rowkey（行键），column key（column family和qualifier）和TimeStamp（时间戳）这个三个维度可以对HBase中的数据进行快速定位。

**HBase中rowkey可以唯一标识一行记录，三种查询方式:**

- 通过get方式，指定rowkey获取唯一一条记录

- 通过scan方式，设置startRow和stopRow参数进行范围匹配

- 全表扫描，即直接扫描整张表中所有行记录

一条数据的唯一标识就是rowkey，那么这条数据存储于哪个分区，取决于rowkey处于哪个一个预分区的区间内，**设计rowkey的主要目的 ，就是让数据均匀的分布于所有的region中，在一定程度上防止数据倾斜**。接下来我们就谈一谈rowkey常用的设计方案。

1. 生成随机数、hash、散列值

   原本rowKey为1001的，SHA1后变成：dd01903921ea24941c26a48f2cec24e0bb0e8cc7

   原本rowKey为3001的，SHA1后变成：49042c54de64a1e9bf0b33e00245660ef92dc7bd

   原本rowKey为5001的，SHA1后变成：7b61dec07e02c188790670af43e717f0f46e8913

   在做此操作之前，一般我们会选择从数据集中抽取样本，来决定什么样的rowKey来Hash后作为每个分区的临界值。

2. 字符串反转

   20170524000001转成10000042507102

   20170524000002转成20000042507102

   这样也可以在一定程度上散列逐步put进来的数据。

3. 字符串拼接

   20170524000001_a12e

   20170524000001_93i7

## 内存优化

HBase操作过程中需要大量的内存开销，毕竟Table是可以缓存在内存中的，**一般会分配整个可用内存的70%给HBase的Java堆**。但是不建议分配非常大的堆内存，因为GC过程持续太久会导致RegionServer处于长期不可用状态，一般16~48G内存就可以了，如果因为框架占用内存过高导致系统内存不足，框架一样会被系统服务拖死。

## **基础优化**

### 允许在HDFS的文件中追加内容

`hdfs-site.xml、hbase-site.xml`

**属性**：dfs.support.append

**解释**：开启HDFS追加同步，可以优秀的配合HBase的数据同步和持久化。默认值为true。

### 优化DataNode允许的最大文件打开数

`hdfs-site.xml`

**属性**：dfs.datanode.max.transfer.threads

**解释**：HBase一般都会同一时间操作大量的文件，根据集群的数量和规模以及数据动作，设置为4096或者更高。默认值：4096

### 优化延迟高的数据操作的等待时间

`hdfs-site.xml`

**属性**：dfs.image.transfer.timeout

**解释**：如果对于某一次数据操作来讲，延迟非常高，socket需要等待更长的时间，建议把该值设置为更大的值（默认60000毫秒），以确保socket不会被timeout掉。

### 优化数据的写入效率

`mapred-site.xml`

**属性**：

mapreduce.map.output.compress

mapreduce.map.output.compress.codec

**解释**：开启这两个数据可以大大提高文件的写入效率，减少写入时间。第一个属性值修改为true，第二个属性值修改为：org.apache.hadoop.io.compress.GzipCodec或者其他压缩方式。

### 设置RPC监听数量

`hbase-site.xml`

**属性**：hbase.regionserver.handler.count

**解释**：默认值为30，用于指定RPC监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。

### 优化HStore文件大小

`hbase-site.xml`

**属性**：hbase.hregion.max.filesize

**解释**：默认值10737418240（10GB），如果需要运行HBase的MR任务，可以减小此值，因为一个region对应一个map任务，如果单个region过大，会导致map任务执行时间过长。该值的意思就是，如果HFile的大小达到这个数值，则这个region会被切分为两个Hfile。

### 优化hbase客户端缓存

`hbase-site.xml`

**属性**：hbase.client.write.buffer

**解释**：用于指定HBase客户端缓存，增大该值可以减少RPC调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少RPC次数的目的。

### 指定scan.next扫描HBase所获取的行数

`hbase-site.xml`

**属性**：hbase.client.scanner.caching

**解释**：用于指定scan.next方法获取的默认行数，值越大，消耗内存越大。

### flush、compact、split机制

当MemStore达到阈值，将Memstore中的数据Flush进Storefile；compact机制则是把flush出来的小文件合并成大的Storefile文件。split则是当Region达到阈值，会把过大的Region一分为二。

**涉及属性：**

即：128M就是Memstore的默认阈值

`hbase.hregion.memstore.flush.size：134217728`

即：这个参数的作用是当单个HRegion内所有的Memstore大小总和超过指定值时，flush该HRegion的所有memstore。RegionServer的flush是通过将请求添加一个队列，模拟生产消费模型来异步处理的。那这里就有一个问题，当队列来不及消费，产生大量积压请求时，可能会导致内存陡增，最坏的情况是触发OOM。

```shell
hbase.regionserver.global.memstore.upperLimit：0.4

hbase.regionserver.global.memstore.lowerLimit：0.38
```



即：当MemStore使用内存总量达到hbase.regionserver.global.memstore.upperLimit指定值时，将会有多个MemStores flush到文件中，MemStore flush 顺序是按照大小降序执行的，直到刷新到MemStore使用内存略小于lowerLimit

