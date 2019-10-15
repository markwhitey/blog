---
title: Interview备忘录
mathjax: false
date: 2019-10-10 09:07:11
tags:
- Interview
categories:
top:
photo:
---



{% cq %}

比较重要的知识点

{% endcq %}

<!-- more -->

<br>

# HDFS 数据安全性如何保证?

---

1. 存储在 HDFS 系统上的文件，会分割成 128M 大小的 block 存储在不同的节点上，block 的副本数默认 3 份，也可配置成更多份；
2. 第一个副本一般放置在与 client（客户端）所在的同一节点上（若客户端无 datanode，则随机放），第二个副本放置到与第一个副本同一机架的不同节点，第三个副本放到不同机架的 datanode 节点，当取用时遵循就近原则；
3. datanode 以 block 为单位，每 3s 报告心跳状态，做 10min 内不报告心跳状态则 namenode 认为 block 已死掉，namonode 会把其上面的数据备份到其他一个 datanode 节点上，保证数据的副本数量；
4. datanode 会默认每小时把自己节点上的所有块状态信息报告给 namenode；
5. 采用 safemode 模式：datanode 会周期性的报告 block 信息。Namenode 会计算 block 的损坏率，当阀值 < 0.999f 时系统会进入安全模式，HDFS 只读不写。 HDFS 元数据采用 secondaryname 备份或者 HA 备份

<br>

​	

# maptask的工作机制

---

> 五个阶段: read map collect spill combine 

## read

map task 通过用户编写的recordreader,从输入的input split 中解析出一个个的key/value

## map

将解析出来的key/value值通过用户自己编写的map函数处理,生成一系列新的key/value值

## collect

在用户编写完map函数中,当数据处理完成后,一般会调用outputCollector.collect输出结果,生成key/value分区,并写入环形内存缓冲区

## spill

当环形区满后,MapReduce会把数据会把数据写入本地磁盘,生成一个临时文件,在此之前需要把数据进行一次本地排序,并在必要的时候进行合并,压缩等操作

(分区内快速排序,key排序)

## combine

把所有的临时文件整合成一个大文件,保证最后每个maptask仅仅生成一个文件<先在每个分区内多轮递归合并



# reducetask的工作机制

---

> copy merge sort reduce

## copy

reducetask从各个maptask上远程拷贝一篇数据,如果大小超过一定阈值



# hadoop 的资源调度方式

FIFO

Capacity Scheduler

Fair Scheduler

LATE

Deadline Scheduler和Constraint-based Scheduler



