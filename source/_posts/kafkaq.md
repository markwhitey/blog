---
title: kafka中遇到的问题
mathjax: false
date: 2019-10-15 10:15:46
tags:
- Big Data
- Kafka
categories:
- Big Data
top:
photo:
---



{% cq %}

对于一个高并发高性能的应用来说，1024 或者 4096 的=文件描述符=限制未免太少，可以适当的调大这个参数。比如使用 ulimit -n 65535 命令将上限提高到 65535，这样足以应对大多数的应用情况

{% endcq %}

<!-- more -->

<br>

Kafka 其实也只是一个 JVM 进程，要想把一个进程搞崩溃，相信大家的 nice idea（骚操作）也不少。本文中只是用了一种很常见的方式来使得 Kafka 的进程崩溃，通过分析崩溃的原因来让我们可以更合理的使用 Kafka。

你可以试着在一台普通的 Linux 机器上创建 10000 个分区的主题。比如下面示例中创建一个主题 topic-bomb：

```shell
[root@node1 kafka_2.11-2.0.0]# bin/kafka-topics.sh --zookeeper localhost:2181/kafka 
--create --topic topic-bomb --replication-factor 1 --partitions 10000
Created topic "topic-bomb".
```

执行完成之后你可以检查一下 Kafka 的进程是否还存在（比如通过 jps 命令或者 ps -aux|grep kafka 命令）。一般情况下，你会发现原本运行完好的 Kafka 服务已经崩溃。此时，你或许会想到，创建这么多个分区，是不是内存不够而引起的进程崩溃，我在启动 Kafka 的时候将 JVM 堆设置的大一点是不是就可以解决了。其实不然，创建这点儿分区而引起的内存增加完全不足以让 Kafka 畏惧。

想要知道真相，我们可以打开 Kafka 的服务日志文件（$KAFKA_HOME/logs/server.log）来一探究竟，你会发现服务日志中出现大量的如下异常：

```shell
[2018-09-13 00:36:40,019] ERROR Error while creating log for topic-bomb-xxx in dir /tmp/kafka-logs (kafka.server.LogDirFailureChannel)
java.io.IOException: Too many open files
	at java.io.UnixFileSystem.createFileExclusively(Native Method)
	at java.io.File.createNewFile(File.java:1012)
	at kafka.log.AbstractIndex.<init>(AbstractIndex.scala:54)
	at kafka.log.OffsetIndex.<init>(OffsetIndex.scala:53)
	at kafka.log.LogSegment$.open(LogSegment.scala:634)
	at kafka.log.Log.loadSegments(Log.scala:503)
	at kafka.log.Log.<init>(Log.scala:237)
```

异常中最关键的信息是：“Too many open flies”，这是一种常见的 Linux 系统错误，通常意味着文件描述符不足，它一般会发生在创建线程、创建 Socket、打开文件这些场景下。在 Linux 系统中的默认设置下，这个文件描述符的个数不是很高，可以通过 ulimit 查看：

```shell
[root@node1 kafka_2.11-2.0.0]# ulimit -n
[root@node1 kafka_2.11-2.0.0]# ulimit -Sn
[root@node1 kafka_2.11-2.0.0]# ulimit -Hn
```

ulimit 是在系统允许的情况下，提供对特定 shell 可利用的资源的控制。（Provides control over the resources avaliable to the shell and to processes started by it, on systems that allow such control）

-H 和 - S 选项设定指定资源的硬限制和软限制。硬限制设定之后不能再添加，而软限制则可以增加到硬限制规定的值。如果 - H 和 - S 选项都没有指定，则软限制和硬限制同时设定。限制值可以是指定资源的数值或者 hard、soft、unlimited 这些特殊值，其中 hard 代表当前硬限制, soft 代表当前软件限制, unlimited 代表不限制。如果不指定限制值, 则打印指定资源的软限制值, 除非指定了 - H 选项。硬限制是可以在任何时候任何进程中设置 但硬限制只能由超级用户提起。软限制是内核实际执行的限制，任何进程都可以将软限制设置为任意小于等于对进程限制的硬限制的值

我们可以通过测试来验证一下本案例中的 Kafka 崩溃是否是由于文件描述符的限制而引起的。首先启动 Kafka 集群，集群中有 3 个节点，配置一样。挑选其中的一台节点 node1 做具体分析，通过 jps 命令我们可以查看到 kafka 的进程 pid 的值：

```shell
[root@node1 kafka_2.11-2.0.0]# jps -l
31796 kafka.Kafka
```

查看当前 Kafka 进程所占用的文件描述符的个数（注意这个值并不是 Kafka 第一次启动时就需要占用的文件描述符的个数，示例中的 Kafka 环境下已经存在了若干主题）：

```
[root@node1 kafka_2.11-2.0.0]# ls /proc/31796/fd | wc -l
```

我们再新建一个只有一个分区的主题，并查看 Kafka 进程所占用的文件描述符的个数：

```
[root@node1 kafka_2.11-2.0.0]# bin/kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic topic-bomb-1 --replication-factor 1 --partitions 1
Created topic "topic-bomb-1".
[root@node1 kafka_2.11-2.0.0]# ls /proc/31796/fd | wc -l
```

可以看到增加了一个分区对应的也只增加了一个文件描述符。
之前我们通过 ulimit 命令可以看到软限制是 1024，不妨我们就创建一个具有 829（1024-195=829）个分区的主题：

```
[root@node1 kafka_2.11-2.0.0]# bin/kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic topic-bomb-2 --replication-factor 1 --partitions 829
Created topic "topic-bomb-2".
[root@node1 kafka_2.11-2.0.0]# ls /proc/31796/fd | wc -l
```

可以看到 Kafka 进程此时占用了 1024 个文件描述符，并且运行完好。这时我们还可以联想到硬限制 4096 这个关键数字，不妨我们再创建一个包含有 3071（4096-1024=3072，这里特定少创建 1 个分区）个分区的主题，示例如下：

```
[root@node1 kafka_2.11-2.0.0]# bin/kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic topic-bomb-3 --replication-factor 1 --partitions 3071
Created topic "topic-bomb-3".
[root@node1 kafka_2.11-2.0.0]# ls /proc/31796/fd | wc -l
```

Kafka 进程依旧完好，文件描述符占用为 4095，逼近最高值 4096。最后我们再次创建一个只有一个分区的主题：

```shell
[root@node1 kafka_2.11-2.0.0]# bin/kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic topic-bomb-4 --replication-factor 1 --partitions 1
Created topic "topic-bomb-4".
[root@node1 kafka_2.11-2.0.0]# ls /proc/31796/fd | wc -l
ls: cannot access /proc/31796/fd: No such file or directory
```

此时 Kafka 已经崩溃，查看进程号时已没有相关信息。查看 Kafka 中的日志，还会发现报错文章开头的异常 “java.io.IOException: Too many open files”，表明已到达上限。

如何避免这种异常情况？对于一个高并发高性能的应用来说，1024 或者 4096 的文件描述符限制未免太少，可以适当的调大这个参数。比如使用 ulimit -n 65535 命令将上限提高到 65535，这样足以应对大多数的应用情况，再高也完全没有必要了。

```
[root@node1 kafka_2.11-2.0.0]# ulimit -n 65535
#可以再次查看一下相应的软硬限制数
[root@node1 kafka_2.11-2.0.0]# ulimit -Hn
[root@node1 kafka_2.11-2.0.0]# ulimit -Sn
```

也可以在 / etc/security/limits.conf 文件中设置，参考如下：

```
#nofile - max number of open file descriptors
root soft nofile 65535
root hard nofile 65535
```

limits.conf 文件修改之后需要重启才能生效。limits.conf 与 ulimit 的区别在于前者是针对所有用户的，而且在任何 shell 都是生效的，即与 shell 无关，而后者只是针对特定用户的当前 shell 的设定。在修改最大文件打开数时，最好使用 limits.conf 文件来修改，通过这个文件，可以定义用户，资源类型，软硬限制等。也可修改 / etc/profile 文件加上 ulimit 的设置语句来是的全局生效。