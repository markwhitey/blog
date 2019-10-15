---
title: MapReduce的Combiner模块
mathjax: false
date: 2019-10-11 11:36:22
tags:
- Big Data
- MapReduce
categories:
- Big Data
top:
photo:
---



{% cq %}

**使用 Combiner 之后传输到 reduce 的数据量有所减少才是 Combiner 存在的意义**

{% endcq %}

<!-- more -->

<br>



# 背景

---

这里要讲的是 MapReduce 的 Combiner 模块，在有的情况下使用 Combiner 会使程序性能提升 N 倍，个人觉得**使用 Combiner 之后传输到 reduce 的数据量有所减少才是 Combiner 存在的意义**。

![](https://i.loli.net/2019/10/13/w1hKD5svIfrcynA.png)

此图只是简要描述 map -> combiner -> reduce 这一过程，最后的 Partitioner 其实就是 Reduce。

<br>

# PS

---

1. 与 mapper 和 reducer 不同的是，combiner 没有默认的实现，需要显式的设置在 conf 中才有作用。

2. 并不是所有的 job 都适用 combiner，只有操作满足结合律的才可设置 combiner。combine 操作类似于：opt(opt(1, 2, 3), opt(4, 5, 6))。如果 opt 为求和、求最大值的话，可以使用，但是如果是求中值的话，不适用。

3. Combiner 在 map 与 reduce 之间，针对每个 key，**有可能会被平用若干次**。

4. **特别值得注意的一点，一个 combiner 只是处理一个结点中的的输出，而不能享受像 reduce 一样的输入（经过了 shuffle 阶段的全量数据）。**

每一个 map 都可能会产生大量的本地输出 ，Combiner 的作用就是对 map 端的输出先做一次合并，以减少在 map 和 reduce 节点之间的数据传输量，以提高网络 IO 性能，是 MapReduce 的一种优化手段之一，其具体的作用如下所述:

（1）Combiner 最基本是实现本地 key 的聚合，对 map 输出的 key 排序，value 进行迭代 。如下所示：

```
map: (K1, V1) → list(K2, V2) 
combine: (K2, list(V2)) → list(K2, V2) 
reduce: (K2, list(V2)) → list(K3, V3)
```

（2）Combiner 还有 本地 reduce 功能（其本质上就是一个 reduce），例如 Hadoop 自带的 wordcount 的例子和找出 value 的最大值的程序，combiner 和 reduce 完全一致，如下所示：

```
map: (K1, V1) → list(K2, V2)
combine: (K2, list(V2)) → list(K3, V3)
reduce: (K3, list(V3)) → list(K4, V4)
```

现在想想，如果在 wordcount 中不用 combiner，那么所有的结果都是 reduce 完成，效率会相对低下。使用 combiner 之后，先完成的 map 会在本地聚合，提升速度。对于 hadoop 自带的 wordcount 的例子，value 就是一个叠加的数字，所以分区完成后就可以进行 reduce 的 value 叠加，而不必要将数据全部 shuffle 到 Reduce 进行处理。



<br>

# 示例

---

下面介绍 Combiner 几种使用的场景：  

- **常用的数学运算（sum、avg、count、min、max 等）**；  
- **自定义逻辑，只要能减少传输到 Redcue 端数据即可**。

注：执行完 Combiner 之后是可以选择执行 Reduce 还是直接输出（Output）的，由驱动端程序指定即可。比如有的程序只是简单的合并下数据，所以，不一定要指定 Reduce。

比如：根据身份证 + 工作地点来统计全国所有城市的总人口数。

根据上面需求，由于中国工作人口最密集的就是北、上、广这几所城市。所以，使用 MapReduce 进行数据统计肯定会出现=数据倾斜=。这时如果在 Reduce 执行之前不进行数据合并（Combiner）则会引发下面两个问题：  

- 网络带宽严重被占降低程序效率（所有数据通过 http shuffle 到 reduce）；  
- 单一节点承载过重降低程序性能。

那么，怎么解决这个问题？

**当然是使用 Combiner，通过 Combiner 对数据进行合并，Combiner 只输出进行过 sum 运算后的记录。最终通过 shuffle 传输到 Reduce 端的数据减少 N 倍，并且 Reduce 需要运算的次数也减少了 N 倍，性能自然就提升了**。代码如下：

## 数据文件

data：

```
0001_370825196902276918
0001_370825196902276919
0002_370825196902276920
0003_370825196902276921
0004_370825196902276922
0004_370825196902276923
0004_370825196902276924
```

## Map 解析数据

```java
public class MyMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

    private final static IntWritable one = new IntWritable(1);

    /**
     * 注意：无重复数据
     */
    public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        // 假设value等于： 城市代码_身份证号码

        String strValue = value.toString();
        int index = strValue.indexOf("_");

        // 由于是统计个数，次数等于1
        context.write(new Text(strValue.substring(0, index)), one);
    }
}
```

注意：保证无重复数据，假设 value 等于（城市代码_身份证号码），由于是统计个数，次数等于 1。

## 根据城市代码分区

```java
/**
 * 
 * @@Description: 根据城市代码分区
 * @@Copyright (c) Pactera All Rights Reserved.
 * @@version 1.0 
 */
public class MyPartitioner extends Partitioner<Text, IntWritable> {

    @Override
    public int getPartition(Text key, IntWritable value, int numPartitions) {
        // 假设key是城市代码
        // value是个数
        return Math.abs(key.hashCode()) % numPartitions;
    }

}
```

> *注：MapReduce 默认就以 key 进行 hashCode 分区，所以此步骤可以省略。但是 key 必须是城市代码。*

## 合并统计

```java
/**
 * 
 * @@Description: sum取和
 * @@Copyright (c) Pactera All Rights Reserved.
 * @@version 1.0 
 */
public class MyCombiner extends Reducer<Text, IntWritable, Text, IntWritable> {
    private IntWritable result = new IntWritable();

    /**
     * Combiner的作用：主要为了合并数据，执行在map
     * -partitioner之后，reduce之前。使用之后传输到reduce的数据量有所减少才是Combiner存在的意义
     * 
     */
    public void reduce(Text key, Iterable<IntWritable> values, Context context)
            throws IOException, InterruptedException {
        int sum = 0;

        // 数据相加取和
        for (IntWritable val : values) {
            sum += val.get();
        }

        result.set(sum);
        context.write(key, result);
    }
}
```

## Main

```java
public static void main(String[] args) throws Exception {

    /**
     * JobConf：map/reduce的job配置类，向hadoop框架描述map-reduce执行的工作
     * 构造方法：JobConf()、JobConf(Class exampleClass)、JobConf(Configuration
     * conf)等
     */
    // 根据自己的实际情况填写输入分析的目录和结果输出的目录
    args = new String[2];

    // args[0] = "hdfs://10.242.157.115:9000/tmp/input";
    // 使用这种方式执行文件，则不需要把core-site.xml和hdfs-site.xml放入src目录，建议将集群环境的配置文件放到src目录
    args[0] = "/tmp/zrh/input";
    args[1] = "/tmp/zrh/output";

    Configuration conf = new Configuration();
    String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
    for (String s : otherArgs) {
        System.out.println(s);
    }

    // 这里需要配置参数即输入和输出的HDFS的文件路径
    if (otherArgs.length != 2) {
        System.err.println("Usage: wordcount <in> <out>");
        System.exit(2);
    }

    // JobConf conf1 = new JobConf(WordCount.class);
    Job job = Job.getInstance(conf);// Job(Configuration conf, String
    job.setJobName("demo"); // job名称

    job.setJarByClass(WordCountTest.class);
    job.setMapperClass(MyMapper.class); // 为job设置Mapper类
    job.setCombinerClass(MyCombiner.class); // 为job设置Combiner类
    job.setReducerClass(MyCombiner.class);

    job.setOutputKeyClass(Text.class); // 设置输出key的类型
    job.setOutputValueClass(IntWritable.class);// 设置输出value的类型
    FileInputFormat.addInputPath(job, new Path(otherArgs[0])); // 为map-reduce任务设置InputFormat实现类
                                                                // 设置输入路径

    FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));// 为map-reduce任务设置OutputFormat实现类
                                                                // 设置输出路径
    System.exit(job.waitForCompletion(true) ? 0 : 1);
}
```

如已经根据[《hadoop2.x-Eclipse 开发环境搭建》](http://www.zhangrenhua.com/2015/12/05/hadoop-eclipse%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/)搭建好本地运行环境，则运行 main 函数启动即可，也可以直接打包放到 hadoop 集群环境中运行。

## 结果

part-r-00000：

```
0001    2
0002    1
0003    1
0004    3
```

Combiner 逻辑与 Reduce 一致所以可以共用。

# 参考资料

- [http://blog.csdn.net/guoery/article/details/8529004](