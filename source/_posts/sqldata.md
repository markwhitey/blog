---
title: SQL--去重问题
mathjax: false
date: 2019-11-24 18:00:04
tags:
- SQL
categories:
- SQL
top:
photo:
---



{% cq %}

无知不代表无罪

{% endcq %}

<!-- more -->

**存在部分字段相同的纪录，即有唯一键主键 ID(最常见情况)**  

如果是这种情况的话用 distinct 是过滤不了的，这就要用到主键 **id** 的唯一性特点及 group by 分组

```sql
select * from table where id in (select max(id) from table group by [去除重复的字段名列表,....])

```

**存在两条完全相同的记录（最简单情况）**

这是最简单的一种情况，用关键字 distinct 就可以去掉

```sql
select distinct id(某一列) from table(表名) where (条件)

```

**没有唯一键 ID**

这种情况我觉得最复杂，目前未使用过，有那位知道其他方法的可以留言，交流一下：

```sql
select identity(int1,1) as id,* into newtable(临时表) from table（原表）
select * from newtable where id in (select max(id) from newtable group by [去除重复的字段名列表,....])
drop table newtable
select identity(int1,1) as id,* into newtable(临时表) from table（原表）
select * from newtable where id in (select max(id) from newtable group by [去除重复的字段名列表,....])
drop table newtable

```