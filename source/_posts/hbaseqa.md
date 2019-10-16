---
title: HBase工具--BulkLoad
mathjax: false
date: 2019-10-16 16:45:01
tags:
categories:
top:
photo:
---



{% cq %}

使用 BulkLoad 是由于实际业务的需要，之前传感器的历史数据是一条一条的插入到 HBase 中，每个月底都需要插入大约 200G 的数据，接近 10 亿条记录。在插入过程中需要占用大量的 IO 资源，极大影响了现有的实时数据入库业务的效率。因此需要寻找一种替代的方案来在不影响现有业务的基础上，不占用大量集群资源，快速高效的插入大量的数据。本文主要介绍了 BulkLoad 的工作机制和应用，以及在使用过程中遇到的问题，最后简要分析了 BulkLoad 的源码步骤。

{% endcq %}

<!-- more -->

<br>

  

BulkLoad 简介
=======================================

---

HBase 可以使用多种方式将数据加载到表中，最直接的方法是使用 MapReduce 作业中的 TableOutputFormat 类，或者使用普通的客户端 API。但是这些并不总是最有效的方法，涉及到的 flush,split,compaction 等操作都容易造成节点不稳定，数据导入慢，耗费资源等问题，在海量数据的导入过程极大的消耗了系统性能。

=BulkLoad 功能使用 MapReduce 作业直接在 HDFS 中生成持久化的 HFile 数据格式文件，然后将生成的 HFile 直接加载到正在运行的集群中，从而完成巨量数据快速入库的操作。配合 MapReduce 完成这样的操作，不占用 Region 资源，不会产生巨量的写入 I/O，所以需要较少的 CPU 和网络资源。=

但是 BulkLoad 也有局限，正常的写入是先写 WAL，然后在写 memstore，而 BulkLoad 没有写 WAL 这一步，因此在 BulkLoad 出现异常情况下，HBase 可无法恢复还未持久化的数据。Train传感器数据要求严格，不能丢任意一条记录。因此 BulkLoad 的局限对其有很大的影响。如果出现数据缺少就要采用其它办法了。

<br>

BulkLoad 的原理和流程
===================================================

---

使用 BulkLoad 需要两个过程：  
1、Transform 阶段：使用 MapReduce 将 HDFS 上的数据生成成 HBase 的底层 Hfile 数据。  
2、Load 阶段：根据生成的目标 HFile，利用 HBase 提供的 BulkLoad 工具将 HFile Load 到 HBase 目录下面。  
![](https://i.loli.net/2019/10/16/qdGRUh8OWIp7MeY.png)

下面使用精简化码来说明整个过程。

## Transform 阶段

编写 Mapper，对数据进行简单的处理生成 rowkey,put 输出  

```java
public static class BulkLoadMapper extends Mapper<LongWritable, Text, ImmutableBytesWritable, Put> {

        @Override

protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

String line = value.toString();

String[] items = line.split(",");

String code = String.join("", items[1].split("\\."));

String row = code + items[2] + items[3] + items[4];

// 生成rowKey

                ImmutableBytesWritable rowKey = new ImmutableBytesWritable(row.getBytes());

                Put put = new Put(Bytes.toBytes(row));   //ROWKEY

// 添加数据

put.addColumn(Bytes.toBytes("cf"), Bytes.toBytes("w"),

                        Bytes.toBytes(line));

                context.write(rowKey, put);

        }

    }
```

## Load 阶段

将 MapReduce 的输出 HFile 加载到 HBase，  

```
public static void loadIncrementalHFileToHBase(final Configuration configuration, final String tableName, final String descPath) throws Exception {

        LoadIncrementalHFiles loder = new LoadIncrementalHFiles(configuration);

        loder.doBulkLoad(new Path(descPath), new HTable(configuration, tableName));

        System.out.println("Bulk Load Completed..");

    }
```

最后需要编写 Job 驱动程序，可以不用设置 reducer，但需要 HFileOutputFormat2.class 来格式化输出。  

```java
public static void bulkLoadDriver(final String tableName, final String srcPath, final String descPath)

            throws Exception {

        Configuration configuration = HBaseConfiguration.create();

Job job = Job.getInstance(configuration);

job.setJarByClass(BulkLoadImport.class);

job.setMapperClass(BulkLoadMapper.class);

// 输出的Key-Value格式

job.setMapOutputKeyClass(ImmutableBytesWritable.class);

job.setMapOutputValueClass(Put.class);

// 使用HFileOutputFormat2.class来格式化输出

job.setOutputFormatClass(HFileOutputFormat2.class);

        HTable table = new HTable(configuration, tableName);

// table.getRegionLocator() 决定了Reducer的数量

        HFileOutputFormat2.configureIncrementalLoad(job, table, table.getRegionLocator());

        FileInputFormat.addInputPath(job, new Path(srcPath));

        FileOutputFormat.setOutputPath(job, new Path(descPath));

// 运行mapreduce后，直接load

if (job.waitForCompletion(true)) {

// 计算处理的记录条数

            Counter counter = job.getCounters().findCounter(TaskCounter.MAP_OUTPUT_RECORDS);

            LOG.info("job finished, total " + counter.getValue() + " records!");

            loadIncrementalHFileToHBase(configuration, tableName, descPath);

        } else {

            System.out.println("jobs run errors");

        }

    }
```

注意点：在建表时，最好进行预分区，通过 HFileOutputFormat2 中代码可以看出，预分区的数量决定了 job 的 Reducer 的数量。因此预分区能够提高 Reducer 的效率。  

```java
List<ImmutableBytesWritable> startKeys = getRegionStartKeys(regionLocator);

LOG.info("Configuring " + startKeys.size() + " reduce partitions " +

"to match current region count");

    job.setNumReduceTasks(startKeys.size());
```

<br>

BulkLoad 源码过程简述
===================================================

----

程序中调用了 LoadIncrementalHFiles 的 doBulkLoad 方法进行 HFile 的移动。其主要流程如下：  
1、初始化一个线程池，设置线程的最大数量  
2、根据参数获取是否对 HFile 的格式进行验证  
3、初始化一个 queue，然后遍历 MapReduce 输出的目录下的所有 HFIles 文件，为每一个 HFile 包装一个 LoadQueueItem，并加入到 queue 中  
4、检查是否有非法的列簇名  
5、遍历队列，尝试将 HFie 加载到一个 region 中，如果失败，它将返回需要重试的 HFie 列表。如果成功，它将返回一个空列表，整个过程是原子性的。  
6、从 RegionServer 中获取到 Region 的名称后，检查是否可以安全的使用 BulkLoad。如果为 False，则使用 ProtobufUtil 的 bulkLoadHFile。否则将使用 SecureBulkLoadClient 的 bulkLoadHFile，将 HFile Load 到 HBase 目录下面。  
7、如果 HFile 的 BulkLoad 失败了，将会尝试将失败的 HFile 将重新移回原来的位置。

**其中需要注意的有**：  
1、当 HFile 的数量极大时，检查 HFile 的格式将会成为最耗时的阶段。可以通过设置`hbase.loadincremental.validate.hfile`来决定是否对 HFile 的格式进行检查（可见 [HBASE-13985](https://issues.apache.org/jira/browse/HBASE-13985)）  
2、BulkLoad 阶段中，采用 Callable 和 Future 实现并发，一但 BulkLoad 失败，HFile 需要重新排队，然后重试。重试次数可以通过`hbase.client.retries.number`进行设置，HBase1.2.5 中默认为 31 次。  
3、BulkLoad 过程结束后，会发现 MapReduce 输出目录下的 HFile 文件都被移走了，说明全部的 HFile 都导入成功。如果想要试验的话，可以先备份一下，免得再跑一边 MapReduce。

<br>

使用问题
====================

---

## MapReducer 阶段为什么会这么慢

1、**建表时设置预分区的个数决定了 Reducer 的数量**。  

```java
List<ImmutableBytesWritable> startKeys = getRegionStartKeys(regionLocator);

LOG.info("Configuring " + startKeys.size() + " reduce partitions " +

"to match current region count");

job.setNumReduceTasks(startKeys.size());
```

2、**数据记录条数过多**，Train传感器的 trans 数据一个月接近 7 亿条数据，对集群的处理能力本身就是一个极大的挑战。  
3、**JVM 参数没有调优**，导致频繁出现 GC，这是后续调优的重点工作。

## Load 阶段为什么这么慢

1、在 Load 阶段阶段中，如果 HFile 文件过多，会触发 hBase 的 compact 和 split 操作。因此 BulkLoad 只是绕过了数据 Put 到 Memstore 和 MemStoreFlush 这个阶段。  
2、当 HFile 的数量极大时，检查 HFile 的格式将会成为最耗时的阶段，可以设置不检查。

## Bulk load operation did not find any files to load in directory xxx

一般是程序写的过程中出现问题，导致 Mapper 阶段没有输出，需要仔细检查

<br>

总结
==============

---

Bulk load 的使用还是需要看场景，对于Train传感器数据来说，使用 Bulk load 的导入效率可能没有直接写来得更快，但是其不占用 Region 资源和大量的 IO 资源，基本上不影响其它业务的运行，还是可以忍受的。

<br>

参考资料
====================

---

[Apache HBase ™ Reference Guide](http://hbase.apache.org/book.html#arch.bulk.load)  
[使用 Bulk Load 快速向 HBase 中导入数据 | JR’s Blog](http://blog.jrwang.me/2015/import-data-to-hbase-using-bulk-loding/)  
[通过 BulkLoad 快速将海量数据导入到 HbaseHadoop 篇 – 过往记忆](https://www.iteblog.com/archives/1889.html#BulkLoad)