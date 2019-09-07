---
title: Big Data框架--Hive笔记
mathjax: false
date: 2019-09-06 09:38:41
tags:
- Big Data
- Hive
categories:
- Big Data
- Hadoop
top:
photo:
---



{% cq %}

Hive是由Facebook开源用于解决海量结构化日志的数据统计。

{% endcq %}

<!-- more -->

<br>

# Hive简介

---

## 什么是Hive

Hive：由Facebook开源用于解决海量结构化日志的数据统计。

**Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。**

本质是：将HQL转化成MapReduce程序

{% mermaid graph TD %}
A[用户创建table] -->|通过映射关系向表中导入数据 |B[数据仓库] 
B --> |metastore中记录着表对应文件的path| C[hive中的元数据库metastore]
C --> |解析器查询输入文件的path| D[hive中的解析器]
D --> F[MapReduce体系结构] 
F --> G(result)
H[用户针对数据表进行数据分析] --> |用户只需要创建表,将表与数据建立映射关系,编写sql分析语句| A
H --> |将 sql语句解析成对应的MapReduce程序,并生成相应的jar包 | D
{% endmermaid %}



**特点:**

- Hive处理的数据存储在HDFS
- Hive分析数据底层的实现是MapReduce
- 执行程序运行在Yarn上

##  Hive的优缺点

### 优点

- 操作接口采用类SQL语法，提供快速开发的能力（简单、容易上手）。
- 避免了去写MapReduce，减少开发人员的学习成本。
- Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合。
- Hive优势在于处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高。
- Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。

### 缺点

- Hive的HQL表达能力有限
  - 迭代式算法无法表达
  - 数据挖掘方面不擅长
- Hive的效率比较低
  - Hive自动生成的MapReduce作业，通常情况下不够智能化
  - Hive调优比较困难，粒度较粗

## Hive的架构原理

- **用户接口：Client**

  CLI（hive shell）、JDBC/ODBC(java访问hive)、WEBUI（浏览器访问hive）

- **元数据：Metastore**

  元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；

  默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore

- **Hadoop**

  使用HDFS进行存储，使用MapReduce进行计算

- **驱动器：Driver**

  - 解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。

  - 编译器（Physical Plan）：将AST编译生成逻辑执行计划。
  - 优化器（Query Optimizer）：对逻辑执行计划进行优化。
  - 执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。

- **运行机制**

  Hive通过给用户提供的一系列交互接口，接收到用户的指令(SQL)，使用自己的Driver，结合元数据(MetaStore)，将这些指令翻译成MapReduce，提交到Hadoop中执行，最后，将执行返回的结果输出到用户交互接口。

## Hive和数据库的比较

### **查询语言**

由于SQL被广泛的应用在数据仓库中，因此，专门针对Hive的特性设计了类SQL的查询语言HQL。熟悉SQL开发的开发者可以很方便的使用Hive进行开发。

### **数据存储位置**

Hive 是建立在 Hadoop 之上的，所有 **Hive 的数据都是存储在 HDFS 中的**。而数据库则可以将数据保存在块设备或者本地文件系统中。

### **数据更新**

由于Hive是针对数据仓库应用设计的，而数据仓库的内容是读多写少的。因此，**Hive中不建议对数据的改写**，所有的数据都是在加载的时候确定好的。而数据库中的数据通常是需要经常进行修改的，因此可以使用 INSERT INTO …  VALUES 添加数据，使用 UPDATE … SET修改数据。

### **索引**

Hive在加载数据的过程中不会对数据进行任何处理，甚至不会对数据进行扫描，因此也没有对数据中的某些Key建立索引。Hive要访问数据中满足条件的特定值时，需要暴力扫描整个数据，因此访问延迟较高。由于 MapReduce 的引入， Hive 可以并行访问数据，因此即使没有索引，对于大数据量的访问，Hive 仍然可以体现出优势。数据库中，通常会针对一个或者几个列建立索引，因此对于少量的特定条件的数据的访问，数据库可以有很高的效率，较低的延迟。由于数据的访问延迟较高，决定了 Hive 不适合在线数据查询。

### **执行**

Hive中大多数查询的执行是通过 Hadoop 提供的 MapReduce 来实现的。而数据库通常有自己的执行引擎。

### **执行延迟**

Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。另外一个导致 Hive 执行延迟高的因素是 MapReduce框架。由于MapReduce 本身具有较高的延迟，因此在利用MapReduce 执行Hive查询时，也会有较高的延迟。相对的，数据库的执行延迟较低。当然，这个低是有条件的，即数据规模较小，当数据规模大到超过数据库的处理能力的时候，Hive的并行计算显然能体现出优势。

### **可扩展性**

由于Hive是建立在Hadoop之上的，因此Hive的可扩展性是和Hadoop的可扩展性是一致的（世界上最大的Hadoop 集群在 Yahoo!，2009年的规模在4000 台节点左右）。而数据库由于 ACID 语义的严格限制，扩展行非常有限。目前最先进的并行数据库 Oracle 在理论上的扩展能力也只有100台左右。

### **数据规模**

由于Hive建立在集群上并可以利用MapReduce进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小。

<br>

# DDL数据定义

---

## 创建数据库

1. 创建一个数据库，数据库在HDFS上的默认存储路径是/user/hive/warehouse/*.db。

   ```mysql
   hive (default)> create database db_hive;
   ```

2. 避免要创建的数据库已经存在错误，增加if not exists判断。（标准写法）

   ```mysql
   hive (default)> create database db_hive;
   FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. Database db_hive already exists
   hive (default)> create database if not exists db_hive;
   ```

   

3. 创建一个数据库，指定数据库在HDFS上存放的位置

   ```mysql
   hive (default)> create database db_hive2 location '/db_hive2.db';
   ```

## 查询数据库

### 显示数据库

1. 显示数据库

   ```mysql
   hive> show databases;
   ```

2. 过滤显示查询的数据库

   ```mysql
   hive> show databases like 'db_hive*';
   OK
   db_hive
   db_hive_1
   ```

### 查看数据库详情

显示数据信息

```mysql
hive> desc database db_hive;
OK
db_hive		hdfs://hadoop102:9000/user/hive/warehouse/db_hive.db	USER	
```

### 修改数据库

用户可以使用ALTER DATABASE命令为某个数据库的DBPROPERTIES设置键-值对属性值，来描述这个数据库的属性信息。**数据库的其他元数据信息都是不可更改的，包括数据库名和数据库所在的目录位置。**

```mysql
hive (default)> alter database db_hive set dbproperties('createtime'='20170830');
```

在Hive中查看修改结果

```mysql
hive> desc database extended db_hive;
db_name comment              location        owner_name            owner_type          parameters
db_hive         hdfs://hadoop102:8020/user/hive/warehouse/db_hive.db     USER   {createtime=20170830}
```

### 删除数据库

```mysql
-- 删除数据库
hive>drop database db_hive2; 

-- 如果删除的数据库不存在，最好采用 if exists判断数据库是否存在
hive> drop database db_hive; 
FAILED: SemanticException [Error 10072]: Database does not exist: db_hive
hive> drop database if exists db_hive2;
hive> drop database db_hive;

-- 如果数据库不为空，可以采用cascade命令，强制删除
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. InvalidOperationException(message:Database db_hive is not empty. One or more tables exist.)
hive> drop database db_hive cascade;
```

## 创建表

### 建表语法

```mysql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name -- CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXISTS 选项来忽略这个异常。
-- EXTERNAL关键字可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION），Hive创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。
[(col_name data_type [COMMENT col_comment], ...)] 

[COMMENT table_comment] -- 为表和列添加注释

[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] -- 创建分区表

[CLUSTERED BY (col_name, col_name, ...) -- 创建分桶表
 
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] -- 不常用

[ROW FORMAT row_format] -- 用户在建表的时候可以自定义SerDe或者使用自带的SerDe。如果没有指定ROW FORMAT 或者ROW FORMAT DELIMITED，将会使用自带的SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的SerDe，Hive通过SerDe确定表的具体的列的数据。SerDe是Serialize/Deserilize的简称，目的是用于序列化和反序列化。

[STORED AS file_format] -- 指定存储文件类型.常用的存储文件类型：SEQUENCEFILE（二进制序列文件）、TEXTFILE（文本）、RCFILE（列式存储格式文件）.如果文件数据是纯文本，可以使用STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCEFILE。
[LOCATION hdfs_path] -- 指定表在HDFS上的存储位置
```

### 管理表

1. 简介

   默认创建的表都是所谓的管理表，有时也被称为内部表。因为这种表，Hive会（或多或少地）控制着数据的生命周期。Hive默认情况下会将这些表的数据存储在由配置项hive.metastore.warehouse.dir(例如，/user/hive/warehouse)所定义的目录的子目录下。**当我们删除一个管理表时，Hive也会删除这个表中数据。管理表不适合和其他工具共享数据。**

2. 基础操作

   ```mysql
   -- 创建普通表
   create table if not exists student2(
id int, name string
   )
   row format delimited fields terminated by '\t'
   stored as textfile
   location '/user/hive/warehouse/student2';
   
   -- 根据查询结果创建表(查询的结果会保存在新的表中)
   create table if not exists student3 as select id, name from student;
   
   -- 查询表的类型
   hive (default)> desc formatted student2;
   Table Type:             MANAGED_TABLE  
   
   ```
   

### 外部表

1. 简介

   因为表是外部表，所以Hive并非认为其完全拥有这份数据。**删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。**

2. 管理表和外部表的使用场景

   每天将收集到的网站日志定期流入HDFS文本文件。**在外部表（原始日志表）的基础上做大量的统计分析**，用到的**中间表、结果表使用内部表存储**，数据通过SELECT+INSERT进入内部表。

3. 基础操作

   ```sql
   -- 创建表
   create external table if not exists default.dept(
   deptno int,
   dname string,
   loc int
   )
   row format delimited fields terminated by '\t';
   
   -- 修改内部表为外部表
   alter table student2 set tblproperties('EXTERNAL'='TRUE');
   
   ```

   > **注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写！**

### 分区表

1. 简介

   分区表实际上就是对应一个HDFS文件系统上的独立的文件夹，该文件夹下是该分区所有的数据文件。Hive中的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过WHERE子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。

2. 基本操作

   ```sql
   -- 引入分区表(需要对日期的部分日志进行区分)
   /user/hive/warehouse/log_partition/20170702/20170702.log
   /user/hive/warehouse/log_partition/20170703/20170703.log
   /user/hive/warehouse/log_partition/20170704/20170704.log
   
   -- 创建分区表
   hive (default)> create table dept_partition(
   deptno int, dname string, loc string
   )
   partitioned by (month string)
   row format delimited fields terminated by '\t';
   
   -- 加载数据到分区中
   hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201709');
   hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201708');
   hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201707’);

   ```

   查询分区表中内容
   
   ```sql
   -- 单分区查询
   hive (default)> select * from dept_partition where month='201709';
   
   -- 多分区联合查询
   hive (default)> select * from dept_partition where month='201709'
                 union
                 select * from dept_partition where month='201708'
                 union
                 select * from dept_partition where month='201707';
   
   _u3.deptno      _u3.dname       _u3.loc _u3.month
   10      ACCOUNTING      NEW YORK        201707
   10      ACCOUNTING      NEW YORK        201708
   10      ACCOUNTING      NEW YORK        201709
   20      RESEARCH        DALLAS  201707
   20      RESEARCH        DALLAS  201708
   20      RESEARCH        DALLAS  201709
   30      SALES   CHICAGO 201707
   30      SALES   CHICAGO 201708
   30      SALES   CHICAGO 201709
   40      OPERATIONS      BOSTON  201707
   40      OPERATIONS      BOSTON  201708
   40      OPERATIONS      BOSTON  201709
   
   -- 增加单个分区
   hive (default)> alter table dept_partition add partition(month='201706') ;
   
   -- 增加多个分区
   hive (default)> alter table dept_partition add partition(month='201705') partition(month='201704');
   
   -- 删除单个分区
   hive (default)> alter table dept_partition drop partition (month='201704');
   
   -- 删除多个分区
   hive (default)> alter table dept_partition drop partition (month='201705'), partition (month='201706');
   
   -- 查看分区表有多少分区
   hive> show partitions dept_partition;
   
   -- 查看分区表结构
   hive> desc formatted dept_partition;
   
   Partition Information          
   col_name              data_type               comment             
   month                   string    
   ```
   
   二级分区表
   
   ```sql
   -- 创建二级分区表
   hive (default)> create table dept_partition2(
                  deptno int, dname string, loc string
                  )
                  partitioned by (month string, day string)
                  row format delimited fields terminated by '\t';
     
   -- 查询分区数据
   hive (default)> select * from dept_partition2 where month='201709' and day='13'
   
   
   ```

<br>

# DSL数据查询

---

## 分桶及抽样查询

1. 简介

   **分区针对的是数据的存储路径；分桶针对的是数据文件。**

   分区提供一个隔离数据和优化查询的便利方式。不过，并非所有的数据集都可形成合理的分区，特别是之前所提到过的要确定合适的划分大小这个疑虑。

   分桶是将数据集分解成更容易管理的若干部分的另一个技术。

2. 数据操作

   ```sql
   -- 创建数据表
   create table stu_buck(id int, name string)
   clustered by(id) 
   into 4 buckets
   row format delimited fields terminated by '\t';
   
   -- 查看表结构
   hive (default)> desc formatted stu_buck;
   Num Buckets:            4     
   
   -- 导入数据到分桶表中
   hive (default)> load data local inpath '/opt/module/datas/student.txt' into table
    stu_buck;
    
    
   -- 设置属性
   hive (default)> set hive.enforce.bucketing=true;
   hive (default)> set mapreduce.job.reduces=-1;
   hive (default)> insert into table stu_buck
   select id, name from stu;
   
   -- 查询分桶数据
   hive (default)> select * from stu_buck;
   OK
   stu_buck.id     stu_buck.name
   1004    ss4
   1008    ss8
   1012    ss12
   1016    ss16
   1001    ss1
   1005    ss5
   1009    ss9
   ```

3. 分桶抽样查询

   **对于非常大的数据集，有时用户需要使用的是一个具有代表性的查询结果而不是全部结果。Hive可以通过对表进行抽样来满足这个需求。**

   查询表stu_buck中的数据。

   ```mysql
   hive (default)> select * from stu_buck tablesample(bucket 1 out of 4 on id);
   ```

   > 注：
   >
   > **tablesample是抽样语句，语法：TABLESAMPLE(BUCKET x OUT OF y) 。**
   >
   > **y必须是table总bucket数的倍数或者因子**。hive根据y的大小，决定抽样的比例。例如，table总共分了4份，当y=2时，抽取(4/2=)2个bucket的数据，当y=8时，抽取(4/8=)1/2个bucket的数据。
   >
   > **x表示从哪个bucket开始抽取，如果需要取多个分区，以后的分区号为当前分区号加上y。**例如，table总bucket数为4，tablesample(bucket 1 out of 2)，表示总共抽取（4/2=）2个bucket的数据，抽取第1(x)个和第3(x+y)个bucket的数据。
   >
   > **注意：x的值必须小于等于y的值**，否则
   >
   > FAILED: SemanticException [Error 10061]: Numerator should not be bigger than denominator in sample clause for table stu_buck

   

## 空字段赋值

1. 函数说明

   NVL：给值为NULL的数据赋值，它的格式是NVL( string1, replace_with)。它的功能是如果string1为NULL，则NVL函数返回replace_with的值，否则返回string1的值，如果两个参数都为NULL ，则返回NULL。

2. 数据操作

   ```sql
   -- 如果员工的comm为NULL，则用-1代替
   hive (default)> select nvl(comm,-1) from emp;
   OK
   _c0
   20.0
   300.0
   500.0
   -1.0
   1400.0
   -1.0
   
   -- 如果员工的comm为NULL，则用领导id代替
   hive (default)> select nvl(comm,mgr) from emp;
   OK
   _c0
   20.0
   300.0
   500.0
   
   ```

## 四个by区别

1. **Sort By**：分区内有序；
2. **Order By**：全局排序，只有一个Reducer；
3. **Distrbute By**：类似MR中Partition，进行分区，结合sort by使用。
4. **Cluster By**：当Distribute by和Sorts by字段相同时，可以使用Cluster by方式。Cluster by除了具有Distribute by的功能外还兼具Sort by的功能。但是排序只能是升序排序，不能指定排序规则为ASC或者DESC。



## 窗口函数

1. **OVER()**：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变化
2. **CURRENT ROW**：当前行
3. **n PRECEDING**：往前n行数据
4. **n FOLLOWING**：往后n行数据
5. **UNBOUNDED**：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING表示到后面的终点
6. **LAG(col,n)**：往前第n行数据
7. **LEAD(col,n)**：往后第n行数据
8. **NTILE(n)**：把有序分区中的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。

## 自定义UDF/UDTF

**在项目中是否自定义过UDF、UDTF函数，以及用他们处理了什么问题，及自定义步骤？**

1. 自定义过。

2. 用UDF函数解析公共字段；用UDTF函数解析事件字段。

**概念:**

- 自定义UDF：继承UDF，重写evaluate方法

- 自定义UDTF：继承自GenericUDTF，重写3个方法：initialize(自定义输出的列名和类型)，process（将结果返回forward(result)），close

**为什么要自定义UDF/UDTF**

因为自定义函数，可以自己埋点Log打印日志，出错或者数据异常，方便调试.

<br>

# Hive调优

---

## MapJoin

如果不指定MapJoin或者不符合MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join，即：在Reduce阶段完成join。容易发生数据倾斜。可以用MapJoin把小表全部加载到内存在map端进行join，避免reducer处理。

## 行列过滤

列处理：在SELECT中，只拿需要的列，如果有，尽量使用分区过滤，少用SELECT *。

行处理：在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤。 

## 合理设置Map数

- **通常情况下，作业会通过input的目录产生一个或者多个map任务。**

  主要的决定因素有：input的文件总个数，input的文件大小，集群设置的文件块大小。

- **是不是map数越多越好？**

  答案是否定的。如果一个任务有很多小文件（远远小于块大小128m），则每个小文件也会被当做一个块，用一个map任务来完成，而一个map任务启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费。而且，同时可执行的map数是受限的。

- **是不是保证每个map处理接近128m的文件块，就高枕无忧了？**

  答案也是不一定。比如有一个127m的文件，正常会用一个map去完成，但这个文件只有一个或者两个小字段，却有几千万的记录，如果map处理的逻辑比较复杂，用一个map任务去做，肯定也比较耗时。

  针对上面的问题2和3，我们需要采取两种方式来解决：即减少map数和增加map数；

## 小文件进行合并

在Map执行前合并小文件，减少Map数：CombineHiveInputFormat具有对小文件进行合并的功能（系统默认的格式）。HiveInputFormat没有对小文件合并功能。

## **合理设置Reduce数**

Reduce个数并不是越多越好

- 过多的启动和初始化Reduce也会消耗时间和资源；

- 另外，有多少个Reduce，就会有多少个输出文件，如果生成了很多个小文件，那么如果这些小文件作为下一个任务的输入，则也会出现小文件过多的问题；

*在设置Reduce个数的时候也需要考虑这两个原则：处理大数据量利用合适的Reduce数；使单个Reduce任务处理数据量大小要合适；*

## **常用参数**

// 输出合并小文件

```sql
SET hive.merge.mapfiles = true; -- 默认true，在map-only任务结束时合并小文件

SET hive.merge.mapredfiles = true; -- 默认false，在map-reduce任务结束时合并小文件

SET hive.merge.size.per.task = 268435456; -- 默认256M

SET hive.merge.smallfiles.avgsize = 16777216; -- 当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge
```

