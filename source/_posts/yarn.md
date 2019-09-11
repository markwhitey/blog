---
title: Big Data框架--Yarn笔记
mathjax: false
date: 2019-09-07 20:06:50
tags:
- Big Data
- Yarn
categories:
- Big Data
top:
photo:
---



{% cq %}

Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。

{% endcq %}

<!-- more -->

<br>

# Yarn架构

---

![](https://i.loli.net/2019/09/07/axLKBfHkeWX6Gig.png)

<br>

# Yarn的工作机制

---

## 图示

![](https://i.loli.net/2019/09/07/ePDfazSbmo98lYW.png)

## 工作机制详解

### **作业提交**

第1步：Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。

第2步：Client向RM申请一个作业id。

第3步：RM给Client返回该job资源的提交路径和作业id。

第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。

第5步：Client提交完资源后，向RM申请运行MrAppMaster。

### **作业初始化**

第6步：当RM收到Client的请求后，将该job添加到容量调度器中。

第7步：某一个空闲的NM领取到该Job。

第8步：该NM创建Container，并产生MRAppmaster。

第9步：下载Client提交的资源到本地。

### **任务分配**

第10步：MrAppMaster向RM申请运行多个MapTask任务资源。

第11步：RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

### **任务运行**

第12步：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

第14步：ReduceTask向MapTask获取相应分区的数据。

第15步：程序运行完毕后，MR会向RM申请注销自己。

### **进度和状态更新**

YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。

### **作业完成**

除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

<br>

# 资源调度器

---

目前，Hadoop作业调度器主要有三种：FIFO、Capacity Scheduler和Fair Scheduler。**Hadoop2.7.2默认的资源调度器是Capacity Scheduler。**

具体设置详见：`yarn-default.xml`文件

```diff
<property>
    <description>The class to use as the resource scheduler.</description>
    <name>yarn.resourcemanager.scheduler.class</name>
+ <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>
```

## 先进先出调度器（FIFO）

![](https://i.loli.net/2019/09/07/kV1O5P8BfhIp6tv.png)

## 容量调度器（Capacity Scheduler）

![](https://i.loli.net/2019/09/07/FGKHDkTBxRNWbji.png)

## 公平调度器（Fair Scheduler）

![](https://i.loli.net/2019/09/07/Uwnc3Koi2WSN5Cu.png)

<br>

# 任务的推测执行

---

## 作业完成时间取决于最慢的任务完成时间

一个作业由若干个Map任务和Reduce任务构成。因硬件老化、软件Bug等，某些任务可能运行非常慢。

思考：系统中有99%的Map任务都完成了，只有少数几个Map老是进度很慢，完不成，怎么办？

## 推测执行机制

发现拖后腿的任务，比如某个任务运行速度远慢于任务平均速度。为拖后腿任务启动一个备份任务，同时运行。谁先运行完，则采用谁的结果。

## 执行推测任务的前提条件

- 每个Task只能有一个备份任务

- 当前Job已完成的Task必须不小于0.05（5%）

- 开启推测执行参数设置。mapred-site.xml文件中默认是打开的。

  ```xml
  <property>
    	<name>mapreduce.map.speculative</name>
    	<value>true</value>
    	<description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
  </property>
  
  <property>
    	<name>mapreduce.reduce.speculative</name>
    	<value>true</value>
    	<description>If true, then multiple instances of some reduce tasks may be executed in parallel.</description>
  </property>
  ```

  

## 不能启用推测执行机制情况

- 任务间存在严重的负载倾斜。
- 特殊任务，比如任务向数据库中写数据。

## 算法原理

![](https://i.loli.net/2019/09/07/vJNPuIO97cUyiEt.png)