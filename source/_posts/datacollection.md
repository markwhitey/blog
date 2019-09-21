---
title: 数据采集中出现的问题以及调优
mathjax: false
date: 2019-09-21 19:47:58
tags:
- Big Data
categories:
- Big Data
- Business Processes
top:
photo:
---



{% cq %}

数据采集中的有关Kafka,Flume,Zookeeper组件调优以及注意

{% endcq %}

<!-- more -->

<br>

# Flume以及Kafka组合组件

----

![](C:\Users\Administrator\Desktop\big data\kAqfO7Kez6bv5Cp.png)

## Flume配置

1. **Source**

   **Taildir Source相比Exec Source、Spooling Directory Source的优势**

   **TailDir Source**：断点续传、多目录。Flume1.6以前需要自己自定义Source记录每次读取文件位置，实现断点续传。

   **Exec Source**可以实时搜集数据，但是在Flume不运行或者Shell命令出错的情况下，数据将会丢失。

   **Spooling Directory Source**监控目录，不支持断点续传。

2. **batchSize大小如何设置？**

   Event 1K左右时，500-1000合适（默认为100）

3. **Channel**

   采用**Kafka** Channel，省去了Sink，提高了效率。

![](C:\Users\Administrator\Desktop\big data\LsHmByjlgDIkpQU.png)

![](C:\Users\Administrator\Desktop\big data\LwnfDGubiTh4QIc.png)

<br>

## Kafka机器数量计算

`Kafka机器数量（经验公式）=2*（峰值生产速度*副本数/100）+1`

**先要预估一天大概产生多少数据，然后用Kafka自带的生产压测（只测试Kafka的写入速度，保证数据不积压），计算出峰值生产速度。再根据设定的副本数，就能预估出需要部署Kafka的数量。**

比如我们采用压力测试测出写入的速度是10M/s一台，峰值的业务数据的速度是50M/s。副本数为2。

Kafka机器数量=2*（50*2/100）+ 1=3台

## Flume内存优化

1. **问题描述**：如果启动消费Flume抛出如下异常

ERROR hdfs.HDFSEventSink: process failed

java.lang.OutOfMemoryError: GC overhead limit exceeded

2. **解决方案步骤**：

   - 在hadoop102服务器的/opt/module/flume/conf/flume-env.sh文件中增加如下配置

     `export JAVA_OPTS="-Xms100m -Xmx2000m -Dcom.sun.management.jmxremote"`

   - 同步配置到hadoop103、hadoop104服务器

     $ xsync flume-env.sh

3. **Flume内存参数设置及优化**

   **JVM heap一般设置为4G或更高，部署在单独的服务器上（4核8线程16G内存）**

   **-Xmx与-Xms最好设置一致，减少内存抖动带来的性能影响，如果设置不一致容易导致频繁fullgc**

## FileChannel和MemoryChannel区别

**MemoryChannel传输数据速度更快**，但因为数据保存在JVM的堆内存中，Agent进程挂掉会导致数据丢失，适用于**对数据质量要求不高的需求**。

FileChannel传输速度相对于Memory慢，但**数据安全保障高**，Agent进程挂掉也可以从失败中恢复数据。

## FileChannel优化

**通过配置dataDirs指向多个路径，每个路径对应不同的硬盘，增大Flume吞吐量**。

官方说明如下：

`Comma separated list of directories for storing log files. Using multiple directories on separate disks can improve file channel peformance`

**checkpointDir和backupCheckpointDir也尽量配置在不同硬盘对应的目录中，保证checkpoint坏掉后，可以快速使用backupCheckpointDir恢复数据**

<br>



# Sink：HDFS Sink

---

## HDFS存入大量小文件，有什么影响？

**元数据层面：**每个小文件都有一份元数据，其中包括文件路径，文件名，所有者，所属组，权限，创建时间等，这些信息都保存在Namenode内存中。所以小文件过多，会占用Namenode服务器大量内存，影响Namenode性能和使用寿命

**计算层面：**默认情况下MR会对每个小文件启用一个Map任务计算，非常影响计算性能。同时也影响磁盘寻址时间。

## HDFS小文件处理

官方默认的这三个参数配置写入HDFS后会产生小文件，``hdfs.rollInterval`、`hdfs.rollSize`、`hdfs.rollCount`

基于以上**hdfs.rollInterval=3600，hdfs.rollSize=134217728，hdfs.rollCount =0，hdfs.roundValue=10，hdfs.roundUnit= second**几个参数综合作用，效果如下：

- tmp文件在达到128M时会滚动生成正式文件

- tmp文件创建超10秒时会滚动生成正式文件

> 举例：在2018-01-01 05:23的时侯sink接收到数据，那会产生如下tmp文件：
>
> /atguigu/20180101/atguigu.201801010520.tmp
>
> 即使文件内容没有达到128M，也会在05:33时滚动生成正式文件



<br>

# Flume拦截器

---

## 拦截器注意事项

项目中自定义了：**ETL拦截器和区分类型拦截器。**

采用两个拦截器的优缺点：

**优点，模块化开发和可移植性；缺点，性能会低一些**

## 自定义拦截器步骤

1. 实现 Interceptor

2. 重写四个方法

   - initialize 初始化

   - public Event intercept(Event event) 处理单个Event
   - public List<Event> intercept(List<Event> events) 处理多个Event，在这个方法中调用Event intercept(Event event)

   - close 方法

3. 静态内部类，实现Interceptor.Builder



<br>

# Hadoop参数调优

---

## HDFS参数调优hdfs-site.xml

- `dfs.namenode.handler.count=20 * log2(Cluster Size)`，比如集群规模为8台时，此参数设置为60

> The number of Namenode RPC server threads that listen to requests from clients. If dfs.namenode.servicerpc-address is not configured then Namenode RPC server threads listen to requests from all nodes.

**NameNode有一个工作线程池，用来处理不同DataNode的并发心跳以及客户端并发的元数据操作。对于大集群或者有大量客户端的集群来说，通常需要增大参数dfs.namenode.handler.count的默认值10。设置该值的一般原则是将其设置为集群大小的自然对数乘以20，即20logN，N为集群大小。**

- 编辑日志存储路径dfs.namenode.edits.dir设置与镜像文件存储路径dfs.namenode.name.dir尽量分开，达到最低写入延迟

## YARN参数调优yarn-site.xml

- **情景描述**：总共7台机器，每天几亿条数据，数据源->Flume->Kafka->HDFS->Hive

  **面临问题**：数据统计主要用HiveSQL，没有数据倾斜，小文件已经做了合并处理，开启的JVM重用，而且IO没有阻塞，内存用了不到50%。但是还是跑的非常慢，而且数据量洪峰过来时，整个集群都会宕掉。基于这种情况有没有优化方案。

- **解决办法**：

  内存利用率不够。这个一般是Yarn的2个配置造成的，单个任务可以申请的最大内存大小，和Hadoop单个节点可用内存大小。调节这两个参数能提高系统内存的利用率。
  - **yarn.nodemanager.resource.memory-mb**

    表示该节点上YARN可使用的物理内存总量，默认是8192（MB），注意，如果你的节点内存资源不够8GB，则需要调减小这个值，而YARN不会智能的探测节点的物理内存总量。

  - yarn.scheduler.maximum-allocation-mb

    单个任务可申请的最多物理内存量，默认是8192（MB）。

- **Hadoop宕机**
  - **如果MR造成系统宕机**。此时要控制Yarn同时运行的任务数，和每个任务申请的最大内存。调整参数：**yarn.scheduler.maximum-allocation-mb**（单个任务可申请的最多物理内存量，默认是8192MB）
  - **如果写入文件过量造成NameNode宕机**。那么**调高Kafka的存储大小**，控制从Kafka到HDFS的写入速度。高峰期的时候用Kafka进行缓存，高峰期过去数据同步会自动跟上。