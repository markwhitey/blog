---
title: 关于DB2数据库的REORG命令的理解以及PENDING状态的处理
mathjax: false
date: 2019-12-21 20:59:54
tags:
- SQL
categories:
- SQL
top:
photo:
---



{% cq %}

## call sysproc.admin_cmd(‘reorg table TABLE_NAME’)

{% endcq %}

<!-- more -->

<br>

# db2暂时挂起状态(pending)

---

**问题： 在DB2数据库中，修改完表的结构时，是否需要对表做一个reorg操作才能使表的状态恢复正常？**

**答：有以下4种操作，需要对表做reorg操作**

1. **SET DATA TYPE altered-data-type (修改字段类型)**

   **但有以下两种情况是例外，不需要reorg：**

   - Increasing the length of a VARCHAR or VARGRAPHIC column       

   - Decreasing the length of a VARCHAR or VARGRAPHIC column without truncating trailing blanks from existing data

2. **SET NOT NULL**
3. **DROP NOT NULL**
4. **DROP COLUMN**

其他的操作，理论上都不需要REORG，但有些操作，是需要REORG之后才能实际生效的，比如**"ALTER TABLE ... COMPRESS YES"，语法上不需要REORG操作，也不会影响表的增删改查操作，但只有REORG之后，才能真正开启压缩**：

After a table has been altered to enable row compression, all rows in the table can be compressed immediately by performing one of the following actions:

1. REORG command   
2. Online table move     
3. Data unload and reload

<br>

# REORG & RUNSTATS

---

当数据库里某张表上有大量插入操作时，需要在表上做RUNSTATS命令保证数据库掌握准确的统计信息。

当数据库里某张表中的记录变化很大时（大量插入、删除、更新操作），需要在表上做REORG和RUNSTATS一组维护操作来优化查询的性能。有的表，可能初始化后从来都不会有数据量变化，就只需要做一次维护；有的表，一天之内的变化就很大，每天需要做多次维护。

注意，针对数据库对象的大量操作，如反覆地删除表，存储过程，会引起系统表中数据的频繁改变，在这种情况下，也要考虑对系统表进行REORG操作。

**一个完整的REORG表的过程应该是由下面的步骤组成的：RUNSTATS -> REORGCHK -> REORG -> RUNSTATS -> BIND或REBIND** 

0. **执行下面命令前要先连接数据库**

1. **RUNSTATS** 由于在第二步中REORGCHK时可以对指定的表进行RUNSTATS操作(在REORGCHK时指定UPDATE STATISTICS)，所以第一步是可以省略的。如果知道哪些特点的表有数据变化，又可以只执行第一步而省略第二步。如果表名为DB2INST1.STAFF，表上有索引，可以执行下面的RUNSTATS操作：db2 runstats on table db2inst1.staff with distribution and detailed indexes all 

2. **REORGCHK** REORGCHK是根据统计公式计算表是否需要重整

   对于每个表有3个统计公式，对索引有5个统计公式(版本8)，如果公式计算结果该表需重整，在输出的REORG字段中相应值为*，否则为-。如果数据库中数据量比较大，在生产系统上要考虑REORGCHK的执行时间可能较长，需安排在非交易时间执行。可以分为对系统表和用户表两部分分别进行REORGCHK：*

   1. 针对系统表进行REORGCHK db2 reorgchk update statistics on table system 使用UPDATE STATISTICS参数指定数据库首先执行RUNSTATS命令。
   2. 针对用户表进行REORGCHK db2 reorgchk update statistics on table user 根据统计公式的计算结果（是否有*），考虑是否必要对表进行REORG。注意，某些小表的结果可能由于统计信息过少而不准确。

3. **REORG** TABLE 执行REORG可以考虑分为表上有索引和没有索引两种情况：

   1. 如果表上有索引如表名为DB2INST1.STAFF，索引名为DB2INST1.STAFF，REORG表：db2 reorg table db2inst1. staff index db2inst1.istaff use tempspace1 建议REORG时可以使用USE参数指定数据重排时使用的临时表空间。如果不指定， REORG工作将会在表所在表空间中原地执行。如果表上有多个索引，INDEX参数值请使用最为重要的索引名。REORG索引：db2 reorg indexes all for table db2inst1.staff 

   2. 如果表上没有索引如表名为DB2INST1.STAFF, SYSIBM.SYSTABLES db2 reorg table db2inst1.staff use tempspace1 db2 reorg table sysibm.systables use tempspace1

 4 **RUNSTATS** 参见步骤1。

5 (可选)上面命令完成后可以重复第二步，检查REORG的结果，如果需要，可以再次执行REORG和RUNSTATS命令。6 BIND或REBIND RUNSTATS命令运行后，应对数据库中的PACKAGE进行重新联编，简单地，可以使用db2rbind命令来完成。例如，如果数据库名为SAMPLE，执行：db2rbind sample -l db2rbind.out 

上述DB2命令详细语法解释需参考: 《Command Reference》

<br>

