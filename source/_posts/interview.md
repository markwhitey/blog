---
title: interview
mathjax: false
date: 2019-10-10 09:07:11
tags:
categories:
top:
photo:
---



{% cq %}



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